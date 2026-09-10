**Cập nhật image trong pipeline CI/CD**

Mục tiêu: tự động cập nhật phiên bản image trong file manifest deployment/rollout để khi có phiên bản image mới, CI/CD pipeline sẽ tự động sử dụng phiên bản image mới nhất đó.

Giải pháp: sử dụng một đoạn script để cập nhật phiên bản image trong file manifest.

Đầu tiên cần thống nhất cách đặt tên cho phiên bản image. Image tag sẽ chuyển sang như sau: IMAGE_TAG: "1.0.$CI_PIPELINE_IID", dựa theo STT của pipeline. Tag sẽ có dạng 1.0.x. Để thực hiện cập nhật tag trong file manifest, có thể tạo 1 stage riêng để thực hiện điều này. Cụ thể stage mới được sử dụng sẽ như sau:

```
update-manifest:
  stage: update
  image: alpine:latest

  before_script:
    - apk add --no-cache git
    - git config user.name "GitLab CI"
    - git config user.email "gitlab-ci@example.com"
    - git remote set-url origin "http://oauth2:${GITLAB_TOKEN}@${CI_SERVER_HOST}/${CI_PROJECT_PATH}.git"

  script:
    - chmod +x scripts/update-rollout.sh
    - ./scripts/update-rollout.sh "$IMAGE_NAME" "$IMAGE_TAG"
    - git add k8s/rollout.yaml
    - git commit -m "update test-app image to $IMAGE_TAG [skip ci]"
    - git push origin HEAD:$CI_COMMIT_REF_NAME
```

- Stage sẽ thực hiện sử dụng script để cập nhật file manifest, rồi commit và push lên git repo.
- Lý do cần cập nhật lên repo git là để Argo CD có thể sync với repo để lấy phiên bản image đúng.
- Trước khi thực hiện script thì cần set origin git. GITLAB_TOKEN là access token của project, cần tạo rồi thêm vào project dưới dạng variable.

Script sử dụng để cập nhật image tag:

```
#!/bin/sh

set -e

IMAGE_NAME="$1"
IMAGE_TAG="$2"

FILE="k8s/rollout.yaml"

if [ -z "$IMAGE_NAME" ] || [ -z "$IMAGE_TAG" ]; then
    echo "Usage: ./update-rollout.sh <image-name> <image-tag>"
    exit 1
fi

echo "Updating image to ${IMAGE_NAME}:${IMAGE_TAG}"

sed -i -E \
  "s#(image:[[:space:]]*)${IMAGE_NAME}:[^[:space:]]+#\1${IMAGE_NAME}:${IMAGE_TAG}#" \
  "$FILE"

echo "Updated image:"
grep "image:" "$FILE"
```

- Script sẽ thực hiện lấy input là image name và image tag đã cung cấp từ stage trước trong pipeline và cập nhật vào trong file manifest.

Sau khi setup script và pipeline thì khi thực hiện commit và push lên git, runner sẽ thực hiện cập nhật file manifest và push thay đổi thêm 1 lần (có tag skip ci để ko thực hiện CI cho lần push này để tránh vòng lặp vô hạn). Khi thành công thì khi sync ở Argo, image mới sẽ được cập nhật.