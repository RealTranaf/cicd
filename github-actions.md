<h1 align="center">Triển khai CI/CD pipeline bằng Github Actions</h1>

Github Actions là dịch vụ CI/CD được tích hợp sẵn trong Github, cho phép tự động hóa các tác vụ như build, test, đóng gói Docker image, deploy lên server hoặc Kubernetes mỗi khi có sự kiện xảy ra trong repo. Các tác vụ này được chạy sử dụng dịch vụ của Github, ko yêu cầu setup nhiều từ người dùng khiên cho đây là một trong những giải pháp CI/CD dễ sử dụng nhất.

Mấu chốt hoạt động của Github Actions nằm ở file workflow, workflow định đoạt toàn bộ pipeline. Những thành phần trong workflow:

- Event: workflow được kích hoạt bởi các event xảy ra với repo như có push mới, pull request, release... Khi các event định nghĩa trong file xảy ra, workflow sẽ chạy.

- Job: workflow gồm nhiều job khác nhau. Các job có thể chạy song song hoặc phụ thuộc vào nhau. VD: build -> test -> deploy.

- Step: trong 1 job sẽ có nhiều bước khác nhau, được Github thực hiện tuần tự. VD: trong job build sẽ có các step checkout code, setup môi trường, cài đặt dependency... Trong các step sẽ chứa các command để yêu cầu pipeline chạy hoặc các action.

- Action: là các module được Github cung cấp cho người dùng để đơn giản hóa một số tác vụ thường sử dụng trong pipeline. VD: để checkout code, có module actions/checkout@v4; setup nodejs có actions/setup-node@v4...

CI/CD pipeline được chạy trên một Runner. Runner là máy thực thi workflow do Github cung cấp hoặc có thể sử dụng runner self-host để chạy trên server riêng.

**Thực hiện triển khai CI/CD bằng Github Actions**

Để tìm hiểu sâu hơn về Actions, thực hiện triển khai pipeline với một ứng dụng frontend đơn giản chạy ReactJS (tạo bằng create-react-app).

File workflow sử dụng để triển khai pipeline: [ci.yaml](ci.yaml). Để Github nhận diện được đúng file pipeline này, cần đặt nó trong thư mục ./.github/workflow của project.

```
name: React CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

- Định nghĩa event của pipeline. Workflow sẽ chạy khi ở nhánh main có yêu cầu push hoặc pull request mới. Mỗi khi có một commit mới được đẩy lên, workflow sẽ chạy.

```
jobs:
  build:
```

- Workflow có định nghĩa 1 job là build.

```
    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build
```

- Yêu cầu runner thực hiện checkout source code, setup môi trường là node và các dependency và build project. Mục đích thực hiện build project ở đây là để kiểm tra xem có lỗi xảy ra trước khi thực hiện build docker image.

```
      - name: Log in to Docker Hub
        if: github.event_name == 'push'
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Build Docker image
        if: github.event_name == 'push'
        run: |
          docker build \
            -t ${{ secrets.DOCKER_USERNAME }}/react-test-app:latest \
            -t ${{ secrets.DOCKER_USERNAME }}/react-test-app:${{ github.sha }} .

      - name: Push Docker image
        if: github.event_name == 'push'
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/react-test-app:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/react-test-app:${{ github.sha }}
```

- Actions sẽ thực hiện đăng nhập vào Docker Hub, thực hiện build Docker image và đẩy image lên một repo trong Docker Hub.

- Cần tạo repo thủ công trước khi đẩy image lên.

- Lý do tạo Docker image là để có thể deploy ứng dụng trong container của Docker hoặc trong Kubernetes (K8s, K3s)

```
      - name: Write kubeconfig
        run: |
          mkdir -p ~/.kube
          cat <<EOF > ~/.kube/config
          ${{ secrets.KUBE_CONFIG }}
          EOF

      - name: Show contexts
        run: |
          kubectl config get-contexts
          kubectl config current-context

      - name: Check cluster
        run: |
          kubectl cluster-info

      - name: Test Cluster
        run: kubectl get nodes

      - name: Update Deployment
        run: |
          kubectl set image deployment/react-test-app \
          react-test-app=${{ secrets.DOCKER_USERNAME }}/react-demo:latest
```
- Cuối cùng, để pipeline có thể tự động deploy ứng dụng, runner sẽ đọc kubectl config do người dùng cung cấp và kết nối với session kubectl ở server host ứng dụng và cập nhật image.

Trong file workflow có sử dụng các biến secret được set trong repo của Github, được bảo mật và mã hóa kỹ càng để ko bị lộ thông tin nhạy cảm.