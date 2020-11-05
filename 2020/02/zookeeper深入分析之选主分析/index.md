# ZooKeeper深入分析之选主分析


ZooKeeper深入分析

<!--more-->

## ZooKeeper核心模块

- 选主逻辑

- 数据一致性保障

- 数据模型

## 节点角色

Server节点组成一个集群，在集群中存在一个唯一的leader节点负责响应写入请求，其他节点只负责接收转发Client请求

Leader：响应写入请求，发起提案，超过半数Follower同意写入，写入成功；
Follower:响应查询，将写入请求发给Leader,参与选举和写入投票
Observer:响应查询，将写入请求发给Leader，不参与投票，只接收写入结果

## 选主逻辑

### 投票

获得法定数量票数：超过一半成为主

### 判断依据

Epoch:leader的任期

ZXID:ZooKeeper事务ID，越大表述数据越新

SID:集群中每个节点的唯一编号

比较策略：任期大的胜出，任期相同比较ZXID大的胜出，ZXID相同比较SID大的胜出


### 逻辑

节点进入Looking状态

广播发起投票

选出最大发起二次投票

超过半数成为主

### 选主代码分析

选主代码主要在FastLeaderElection类中的lookForLeader方法

```java
public Vote lookForLeader() throws InterruptedException {
    try {
        self.jmxLeaderElectionBean = new LeaderElectionBean();
        MBeanRegistry.getInstance().register(self.jmxLeaderElectionBean, self.jmxLocalPeerBean);
    } catch (Exception e) {
        LOG.warn("Failed to register with JMX", e);
        self.jmxLeaderElectionBean = null;
    }

    self.start_fle = Time.currentElapsedTime();
    try {
        // 存储选举投票
        Map<Long, Vote> recvset = new HashMap<Long, Vote>();

        /*
            * The votes from previous leader elections, as well as the votes from the current leader election are
            * stored in outofelection. Note that notifications in a LOOKING state are not stored in outofelection.
            * Only FOLLOWING or LEADING notifications are stored in outofelection. The current participant could use
            * outofelection to learn which participant is the leader if it arrives late (i.e., higher logicalclock than
            * the electionEpoch of the received notifications) in a leader election.
            */
        // 当
        Map<Long, Vote> outofelection = new HashMap<Long, Vote>();

        int notTimeout = minNotificationInterval;

        synchronized (this) {
            // 选举轮数加一
            logicalclock.incrementAndGet();
            // 初始化投票信息
            updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch());
        }

        LOG.info(
            "New election. My id = {}, proposed zxid=0x{}",
            self.getId(),
            Long.toHexString(proposedZxid));
        // 发送广播，发送投票信息
        sendNotifications();

        SyncedLearnerTracker voteSet;
        // 循环接收投票信息，直到选出一个leader
        while ((self.getPeerState() == ServerState.LOOKING) && (!stop)) {
            // 从接收到的投票队列中获取投票信息
            // recvqueue LinkedBlockingQueue<Notification> recvqueue
            Notification n = recvqueue.poll(notTimeout, TimeUnit.MILLISECONDS);

            // 判断是否有投票信息，没有的话判断其他投票人是否已经投票
            if (n == null) {
                // 如果其他参与者都已经投票
                if (manager.haveDelivered()) {
                    // 发送自身的投票信息
                    sendNotifications();
                } else {
                    // 没有投票的话可能是连接断开了，重新连接一下
                    manager.connectAll();
                }
                // 延长超时时间
                int tmpTimeOut = notTimeout * 2;
                notTimeout = Math.min(tmpTimeOut, maxNotificationInterval);
                LOG.info("Notification time out: {}", notTimeout);
            } else if (validVoter(n.sid) && validVoter(n.leader)) {
                // 如果获取到了投票信息，校验投票信息是否合法
                // validVoter(n.sid) 校验的是投票者的sid
                /*
                    * Only proceed if the vote comes from a replica in the current or next
                    * voting view for a replica in the current or next voting view.
                    */
                switch (n.state) {
                case LOOKING:
                    if (getInitLastLoggedZxid() == -1) {
                        LOG.debug("Ignoring notification as our zxid is -1");
                        break;
                    }
                    if (n.zxid == -1) {
                        LOG.debug("Ignoring notification from member with -1 zxid {}", n.sid);
                        break;
                    }
                    // If notification > current, replace and send messages out
                    if (n.electionEpoch > logicalclock.get()) {
                        logicalclock.set(n.electionEpoch);
                        recvset.clear();
                        if (totalOrderPredicate(n.leader, n.zxid, n.peerEpoch, getInitId(), getInitLastLoggedZxid(), getPeerEpoch())) {
                            updateProposal(n.leader, n.zxid, n.peerEpoch);
                        } else {
                            updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch());
                        }
                        sendNotifications();
                    } else if (n.electionEpoch < logicalclock.get()) {
                            LOG.debug(
                                "Notification election epoch is smaller than logicalclock. n.electionEpoch = 0x{}, logicalclock=0x{}",
                                Long.toHexString(n.electionEpoch),
                                Long.toHexString(logicalclock.get()));
                        break;
                    } else if (totalOrderPredicate(n.leader, n.zxid, n.peerEpoch, proposedLeader, proposedZxid, proposedEpoch)) {
                        updateProposal(n.leader, n.zxid, n.peerEpoch);
                        sendNotifications();
                    }

                    LOG.debug(
                        "Adding vote: from={}, proposed leader={}, proposed zxid=0x{}, proposed election epoch=0x{}",
                        n.sid,
                        n.leader,
                        Long.toHexString(n.zxid),
                        Long.toHexString(n.electionEpoch));

                    // don't care about the version if it's in LOOKING state
                    recvset.put(n.sid, new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch));

                    voteSet = getVoteTracker(recvset, new Vote(proposedLeader, proposedZxid, logicalclock.get(), proposedEpoch));

                    if (voteSet.hasAllQuorums()) {

                        // Verify if there is any change in the proposed leader
                        while ((n = recvqueue.poll(finalizeWait, TimeUnit.MILLISECONDS)) != null) {
                            if (totalOrderPredicate(n.leader, n.zxid, n.peerEpoch, proposedLeader, proposedZxid, proposedEpoch)) {
                                recvqueue.put(n);
                                break;
                            }
                        }

                        /*
                            * This predicate is true once we don't read any new
                            * relevant message from the reception queue
                            */
                        if (n == null) {
                            setPeerState(proposedLeader, voteSet);
                            Vote endVote = new Vote(proposedLeader, proposedZxid, logicalclock.get(), proposedEpoch);
                            leaveInstance(endVote);
                            return endVote;
                        }
                    }
                    break;
                case OBSERVING:
                    LOG.debug("Notification from observer: {}", n.sid);
                    break;
                case FOLLOWING:
                case LEADING:
                    /*
                        * Consider all notifications from the same epoch
                        * together.
                        */
                    if (n.electionEpoch == logicalclock.get()) {
                        recvset.put(n.sid, new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch, n.state));
                        voteSet = getVoteTracker(recvset, new Vote(n.version, n.leader, n.zxid, n.electionEpoch, n.peerEpoch, n.state));
                        if (voteSet.hasAllQuorums() && checkLeader(recvset, n.leader, n.electionEpoch)) {
                            setPeerState(n.leader, voteSet);
                            Vote endVote = new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch);
                            leaveInstance(endVote);
                            return endVote;
                        }
                    }

                    /*
                        * Before joining an established ensemble, verify that
                        * a majority are following the same leader.
                        *
                        * Note that the outofelection map also stores votes from the current leader election.
                        * See ZOOKEEPER-1732 for more information.
                        */
                    outofelection.put(n.sid, new Vote(n.version, n.leader, n.zxid, n.electionEpoch, n.peerEpoch, n.state));
                    voteSet = getVoteTracker(outofelection, new Vote(n.version, n.leader, n.zxid, n.electionEpoch, n.peerEpoch, n.state));

                    if (voteSet.hasAllQuorums() && checkLeader(outofelection, n.leader, n.electionEpoch)) {
                        synchronized (this) {
                            logicalclock.set(n.electionEpoch);
                            setPeerState(n.leader, voteSet);
                        }
                        Vote endVote = new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch);
                        leaveInstance(endVote);
                        return endVote;
                    }
                    break;
                default:
                    LOG.warn("Notification state unrecoginized: {} (n.state), {}(n.sid)", n.state, n.sid);
                    break;
                }
            } else {
                if (!validVoter(n.leader)) {
                    LOG.warn("Ignoring notification for non-cluster member sid {} from sid {}", n.leader, n.sid);
                }
                if (!validVoter(n.sid)) {
                    LOG.warn("Ignoring notification for sid {} from non-quorum member sid {}", n.leader, n.sid);
                }
            }
        }
        return null;
    } finally {
        try {
            if (self.jmxLeaderElectionBean != null) {
                MBeanRegistry.getInstance().unregister(self.jmxLeaderElectionBean);
            }
        } catch (Exception e) {
            LOG.warn("Failed to unregister with JMX", e);
        }
        self.jmxLeaderElectionBean = null;
        LOG.debug("Number of connection processing threads: {}", manager.getConnectionThreadCount());
    }
}
```