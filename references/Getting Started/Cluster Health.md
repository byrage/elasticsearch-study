## Cluster Health

- curl이나 HTTP/REST를 통해서 클러스터가 어떻게 동작하고 있는지 체크할 수 있다.
- 클러스터 헬스체크를 하기 위해서 \_cat API를 사용할 것. 레퍼런스 문서에서 제공하는 COPY AS CURL을 이용하여 터미널에서 실행할 수 있고, VIEW IN CONSOLE을 이용하여 키바나 콘솔에서 손쉽게 확인할 수도 있다.
- 키바나 콘솔을 사용하기 위해서는 로컬에 kibana를 설치하고 실행중인 상태여야 함

### /\_cat/health?v
```
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1475247709 17:01:49  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```
- elasticsearch가 green status임을 확인 할 수 있음.

#### Status
1. Green : everything is good (cluster is fully functional)
- Yellow : 모든 데이터가 사용가능하지만, 몇몇의 replicas가 아직 배치되지 않음. (cluster is fully functional)
- Red : 몇몇의 데이터가 사용가능하지 않음. (cluster is partially functional)
> Note : 클러스터 상태가 Red일때에도 사용가능한 샤드에서 계속해서 검색이 가능하지만, 할당되지 않은 샤드가 있기 때문에 가능한 빨리 고쳐야 합니다.

위의 response로 1개의 노드와 0개의 샤드를 볼 수 있음.
엘라스틱 서치는 ** unicast network discovery**를 사용하기때문에, 우연히 같은 컴퓨터에서 1개 이상의 노드가 실행되고 하나의 클러스터에 join 할 수도 있다.
이런 경우에는 위의 response에서 1개 이상의 노드를 보게 될 것이다.

### /\_cat/nodes?v
```
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           10           5   5    4.46                        mdi      *      PB2SGZY
```
- 클러스터에서 작동하는 노드의 이름을 확인할 수 있다.
