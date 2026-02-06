# migros-one-devops-case

## Dosyalar

kind-config.yaml : Lokalimizde çalıştıracağımız KIND cluster'ının konfigürasyon dosyasıdır. 80 ve 443 portları için port forwarding ve NGINX ingress için gereken label verilmiştir.

deployment.yaml : Ayağa kaldıracağımız 2048 oyununun manifest dosyasıdır. İmaj olarak sanoobtv/2048 kullanılmaktadır. 1 replika olacak ve container portu 80 olacak şekilde konfigüre edilmiştir.

service.yaml : 2048 oyun pod'unu erişilebilir bir ClusterIP servis olacak şekilde oluşturacak manifest dosyasıdır.

ingress.yaml : 2048 oyununun servisini ve Grafana servisini dışarıya sırasıyla 2048.local ve grafana.local olacak şekilde 80 portunu kullanarak açmaktadır.

servicemonitor.yaml : Oyunu içeren podu ve endpointini prometheus'un nasıl takip edebileceğini belirten monitoring konfigürasyonudur.

## Adımlar

`kind create cluster --name migros-2048 --config kind-config.yaml`

Konfig dosyasındaki KIND cluster'ını oluşturuyoruz.

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml`

KIND ile uyumlu olan NGINX ingress'i cluster'ımıza dahil ediyoruz

`kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller`

Normal lokalimde oluştururken wait komutu kullanmadım fakat script halinde çalıştıracaksak ingress-nginx podları hazır hale gelene kadar ingressleri oluşturamayız.

```
kubectl create namespace game --dry-run=client -o yaml | kubectl apply -f -
kubectl create namespace monitoring --dry-run=client -o yaml | kubectl apply -f -
```

Oyun için ve monitoring komponentleri için ayrı iki namespace oluşturuyoruz.

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring
```

Helm chartları kullanarak kubernetes prometheus stack'i cluster'ımız içerisinde ayağa kaldırıyoruz. Bunun içerisinde Prometheus, Grafana ve AlertManager bulunmakta.

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl apply -f servicemonitor.yaml
```

Oyunumuz için gereken deployment, servis ve ingress'i ayağa kaldırıyoruz. Monitoring için ingress dosyası içerisinde grafana için bir çıkış noktası daha ayağa kaldırıyoruz ve service monitor ile de prometheus'un oyunumuzu monitor edebilmesi için gereken servicemonitor'u oluşturuyoruz.

```
grep -q "2048.local" /etc/hosts || echo "127.0.0.1 2048.local" | sudo tee -a /etc/hosts
grep -q "grafana.local" /etc/hosts || echo "127.0.0.1 grafana.local" | sudo tee -a /etc/hosts
```

Lokalimizde DNS kaydı olarak /etc/hosts dosyamıza 2048.local ve grafana.local kayıtlarını oluşturuyoruz. Eğer bu kayıtlar zaten varsa bir aksiyon alınmıyor.

```
echo "Grafana Password:"
kubectl get secret --namespace monitoring -l app.kubernetes.io/component=admin-secret -o jsonpath="{.items[0].data.admin-password}" | base64 --decode ; echo
```

Grafana ayağa kaldırıldığında rastgele bir şifre ile ayağa kalkıyor. Şifreye bu komutun çıktısıyla giriş yapabiliyoruz default admin kullanıcısına.
