# Coolify Github Actions

- [github actions for coolify](https://coolify.io/docs/knowledge-base/git/github/github-actions/)


```yml
# For more details, read this: https://coolify.io/docs/github-actions
name: Build Static Image
on:
  push:
    branches: ["main"]
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: "andrasbacsai/github-actions-with-coolify"

jobs:
  amd64:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v3
      - name: Login to ghcr.io
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.TOKEN  }}
      - name: Build image and push to registry
        uses: docker/build-push-action@v4
        with:
          context: .
          file: Dockerfile
          platforms: linux/amd64
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
      - name: Deploy to Coolify
        run: | 
         curl --request GET '${{ secrets.COOLIFY_WEBHOOK }}' --header 'Authorization: Bearer ${{ secrets.COOLIFY_TOKEN }}'
```



> https://github.com/andrasbacsai/github-actions-with-coolify/blob/main/.github/workflows/build.yaml