# Helm chart

Helm là 1 trình quản lý gói ứng dụng cho k8s, điều phối tải xuống, cài đặt và triển khai ứng dụng. Helm chart là cách để bạn xác định 1 ứng dụng bao gồm các tài nguyên nào trong k8s.

Helm giúp quản lý việc triển khai các ứng dụng dễ dàng hơn trong k8s bằng cách sử dụng các template được làm sẵn. Tất cả Helm đều có cấu trúc giống nhau nhưng có thể linh hoạt chạy bất cứ ứng dụng nào trên k8s

## Lợi ích của Helm Chart

Mục đích của nó là làm cho toàn bộ quá trình thiết lập việc triển khai 1 ứng dụng hoặc 1 môi trường cho nhiều mục đích trở nên dễ dàng hơn. Nhiều tác vụ thủ công có thể hoàn toàn tự động, giúp đơn giản hóa việc sử dụng k8s hơn.

Helm linh hoạt dựa vào file value.yaml. Ta vào dile value.yaml để dễ dàng thực hiện việc bảo trì

## Cài đặt Helm CLI

```
curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

Check version
```
helm version --short
```

Tải các kho lưu chữ Chart
```
helm repo add stable https://charts.helm.sh/stable
```

## Tự tạo chart nginx và deploy chúng

### Create chart

```
helm create example-project
```

Cấu trúc thư mục như sau:
```
/example-project
    /Chart.yaml # mô tả của chart
    /values.yaml # các giá trị mặc định, chúng ta có thể thay đổi trong khi cài đặt hay nâng cấp project của chúng ta
    /charts/ # subcharts
    /templates/ # template file
```

### Tùy chỉnh lại theo ý mình

Nội dung trong file `Template`:
- `deployment.yaml`: deployment với kubernetes
- `_helpers.tpl`: Mẫu có sẵn có thể tái sử dụng trong chart
- `ingress.yaml`: Liệt kê triển khai các quyền để ứng dụng của bạn truy cập được k8s
- `serviceaccount.yaml`: Tạo tài khoản cho service mình dùng
- `service.yaml`: triển khai các deployment với các service
- `tests/`: Thư mục chưa các testing về chart

Để tạo chart từ đầu và ta xóa các file:
```
rm -rf example-project/templates/
rm -rf example-project/Chart.yaml
rm -rf example-project/values.yaml
```

Mô tả file `Chart.yaml`:
```
apiVersion: v2
name: example-project
description: A Helm chart for Kubernetes Microservices application
version: 0.1.0
appVersion: 1.0
```

Tạo folder `deployment` và `service` bên trong folder template:
```
mkdir -p example-project/templates/deployment
mkdir -p example-project/templates/service
```

Tạo file `coffee-deployment.yaml`:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coffee
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: coffee
  template:
    metadata:
      labels:
        app: coffee
    spec:
      containers:
      - name: coffee
        image: {{ .Values.hello.image }}:{{ .Values.version }}
        ports:
        - containerPort: 80
```

Có 1 chút khác biệt so với deployment thông thường là có config {{ .Values.relicas }} hay {{ .Values.hello.image }}:{{ .Values.version }} được maping tới `values.yaml`

Tạo file `coffee-service.yaml`:
```
apiVersion: v1
kind: Service
metadata:
  name: coffee-svc
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: coffee
```

Tạo file `values.yaml`
```
# Default values for example-project.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

# Release-wide Values
replicas: 3
version: 'plain-text'

# Service Specific Values
hello:
  image: nginxdemos/hello
```

### Deploy example-project Chart

Trước khi deployment ta sẽ test Chart để xác nhận các config của ta là chính xác:
```
helm install --debug --dry-run example-project example-project
```

Deploy:
```
helm install example-project example-project
```

### Xóa example-project

```
helm uninstall example-project
```