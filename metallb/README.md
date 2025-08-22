# Metal LB

- bgpを使用した構成です。
  - ix2215の場合は以下のようなコンフィグを流す (ほかのix/ciscoでもそのままか少しいじれば動くはず)

```text
router bgp 65001
  router-id 192.168.1.1
  address-family ipv4 unicast
    network 192.168.1.0/24
  peer-group k8scluster01 remote-as 65002
    neighbor 192.168.1.81
    neighbor 192.168.1.94
!
```

- ipアドレスは環境に応じて変えること
- neighborのipはワーカーノードのipを指定
