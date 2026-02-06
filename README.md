# migros-one-devops-case

## Dosyalar

kind-config.yaml : Lokalimizde çalıştıracağımız KIND cluster'ının konfigürasyon dosyasıdır. 80 ve 443 portları için port forwarding ve NGINX ingress için gereken label verilmiştir.

deployment.yaml : Ayağa kaldıracağımız 2048 oyununun manifest dosyasıdır. İmaj olarak sanoobtv/2048 kullanılmaktadır. 1 replika olacak ve container portu 80 olacak şekilde konfigüre edilmiştir.

service.yaml : 2048 oyun pod'unu erişilebilir bir ClusterIP servis olacak şekilde oluşturacak manifest dosyasıdır.

ingress.yaml : 2048 oyununun servisini ve Grafana servisini dışarıya sırasıyla 2048.local ve grafana.local olacak şekilde 80 portunu kullanarak açmaktadır.

servicemonitor.yaml : Oyunu içeren podu ve endpointini prometheus'un nasıl takip edebileceğini belirten monitoring konfigürasyonudur.

## Adımlar

