# HPA - Horizontal Pod Autoscaling

## Auto Scaling Pod là gì ?

Là quá trình thực hiện tăng số lượng pod trong 1 node lên 1 số lượng đã được định sẵn. Quá trình này được thực hiện khi xảy ra 1 hoặc nhiều sự kiện, VD: CPU đạt trên 70%, số lượng request đến server lớn hơn 500 req/s

## Auto Scaling để làm gì ?

HPA đem lại lợi ích: kinh tế, tự động hóa việc tăng giảm cấu hình hệ thống phù hợp với các hệ thống có khối lượng tải (mức enduser) biến đổi nhiều và khó dự đoán

So với mô hình "truyền thống" kiểu fix cứng số lượng các Pods, auto scaling thích ứng để phù hợp với nhu cầu sử dụng. VD: khi lượng truy cập vào hệ thống vào buổi đêm giảm xuống, các Pods có thể được set vào sleep mode (sleep mode để dễ dàng bật lại ngay đối phó với sự tăng bất lường từ lượng truy cập)

## Metrics Server

Là 1 server tập hợp lại các metrics (chỉ số đo lường) của các container(các Pods) phục vụ cho chu trình autoscaling tích hợp trong k8s

<img src="./Images/metric.png" alt="metric" width="800" />

Các bước trên sơ đồ:
1. Các metrics (mức sử dụng RAM, CPU) được thu thập từ các Pods
2. Các metrics này sẽ được đẩy đến `kubelet`
3. Metrics Server thu thập metrics qua `kubelet`
4. Metrics được đẩy đến API server, HPA sẽ gọi API này để lấy các metrics, tính toán để scale pods

## Ưu điểm

- Tích hợp sẵn nên tối ưu nhất cho các thành phần k8s
- Thực hiện scale nhanh chóng

## Nhược điểm

- Chỉ hỗ trợ 1 vài metric cơ bản (CPU, ram, ...)

Cài đặt Metrics Server:
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Kiểm tra trạng thái service vừa tải về:
```
kubectl -n kube-system get pod
```

<img src="./Images/ktra-metric.png" alt="ktra-metric" width="800" />

Kiểm tra describe pod:
```
kubectl -n kube-system describe pod <metrics-server....>
```
<img src="./Images/describe-pod-metrics.png" alt="describe-pod-metrics" width="800" />

Sửa lại metrics-server:
```
kubectl -n kube-system edit deploy metrics-server
```
Thêm phần command vào file.
```
containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        command:
        - /metrics-server
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
```

Kiểm tra lại bằng câu lệnh:
```
kubectl top node
```

## Cluster Auto-Scaler

Khi `ban điều hành` HPA tăng số lượng Pod, thì node cũng cần phải tăng lên để đáp ứng số Pod mới này.

`Cluster Auto-Scaler` là một chức năng trong K8s, chịu trách nghiệm tăng/giảm số lượng của node sao cho phù hợp với số lượng Pods vận hành

`Cluster Auto-Scaler` sẽ điều chỉnh tự động kích thước của Kubernetes cluster (hay số nodes) khi một trong các điều kiện sau thỏa mãn:
- Một số Pods run bị fail trong cluster do lý do không đủ tài nguyên
- Có node trong cluster không được sử dụng hết công suất, các Pods của nó có thể vận hành được trên các node khác mà có tài nguyên đang dư giả

<img src="./Images/cluster-autoscaler.png" alt="describe-pod-metrics" width="800" />

## Cấu hình Autoscanling với HPA

Tạo 1 deployment gồm 1 Pod chạy Apache và 1 service trỏ tới Pod đó. Sau đó cấu hình HPA cho Deployment đó với điều kiện tải CPU của Pod > 50% thì sẽ scale số pod lên, và khi tải giảm xuống thì số Pod cũng được giảm theo.

Tạo file deployment php-apache.yaml như sau:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 2
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```

Apply file để tạo deployment và service:
```
kubectl apply -f php-apache.yaml
```

Cấu hình HPA cho deployment này để khi các Pod tăng tải sẽ được autoscale:
```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
kubectl get hpa
```

Ta chạy 1 Pod gọi liên tục tới php-apache. Sử dụng 2 màn hình terminal, 1 màn hình để chạy pod để generate load, màn hình còn lại để theo dõi trạng thái của deployment để xem autoscale như thế nào
```
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

Màn hình còn lại:
```
kubectl get hpa php-apache --watch
```

## Sử dụng KEDA để auto scale pod k8s

KEDA - Kubernetes-based Event Driven Autoscaler.

### KEDA hoạt động như thế nào?

Hoạt động với vai trò như 1 metric server biên dịch các metric nhận được từ external server về cấu trúc của HPA có thể hiểu được để tiến hành Scale qua HPA.

KEDA có 1 khái niệm ScaledObject, khi tạo ScaleObject cũng sẽ tạo thêm 1 HPA để thực hiện việc scale pod.

Cài đặt bằng Helm :
```
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
kubectl create namespace keda
helm install keda kedacore/keda --namespace keda
```

Cài đặt bằng yaml file:
```
kubectl apply -f https://github.com/kedacore/keda/releases/download/v2.4.0/keda-2.4.0.yaml
```

Tạo Scaled Object:
```
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: prometheus-scale
  namespace: default
spec:
  scaleTargetRef:
    name: appdeploy (1)
  minReplicaCount: 3 (2)
  maxReplicaCount: 10 (3)
  triggers:
  - type: prometheus (4)
    metadata:
      serverAddress: http://103.56.156.199:9090/ (5)
      metricName: total_http_request (6)
      threshold: '60' (7)
      query: sum(irate(by_path_counter_total{}[60s])) (8)
```

(1): Tên Deployment muốn scale

(2): Số lượng pod nhỏ nhất

(3): Số lượng pod lớn nhất

(4): Kiểu external Server

(5): Địa chỉ external Server

(6): Tên metric

(7): Giá trị sẽ gọi event (ở đây nếu số req/s > 60)

(8): Câu query lấy dữ liệu từ external Server

### Ưu điểm

- Hỗ trợ nhiều loại metrics khác nhau

### Nhược điểm

- Sẽ tốn 1 phần tài nguyên để chạy các thành phần của KEDA.