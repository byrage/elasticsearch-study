# G1 GC

- JVM Heap 영역을 Region이라고 하는 2048/힙 사이즈로 분할한다. 즉, 8G의 Heap이라면 하나의 Region의 크기는 4MB(8192MB/2048 = 4MB)
- Region의 크기는 조정가능(1MB~32MB)하지만 권장하지 않는다.(-XX:G1RegionSize)
- 각 Region은 기존의 JVM heap 영역이던, New Area(Eden Area), Survivor Area, Old Generation Area, Permanent Generation Area를 담당하지만, 동적으로 역할이 변경 될 수 있다.

<img src="http://cfile6.uf.tistory.com/image/2216DD4355F22A82320356">
<img src="http://cfile2.uf.tistory.com/image/22778D3E55F2310D14F298">
