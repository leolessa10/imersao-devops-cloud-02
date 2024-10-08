AWS
name: CI-CD

on:
  push:
    branches: ["main"]
  workflow_dispatch:
jobs:
  CI:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do repositorio
        uses: actions/checkout@v4.1.1
      - name: Docker Login
        uses: docker/login-action@v3.0.0
        with:
          username: ${{secrets.DOCKERHUB_USER}}
          password: ${{secrets.DOCKERHUB_PWD}}
      - name: Build and push Docker images
        uses: docker/build-push-action@v5.0.0
        with:
          context: ./kube-news/src
          file: ./kube-news/src/Dockerfile
          push: true
          tags: |
            leolessa10/kube-news:${{ github.run_number }}
            leolessa10/kube-news:latest
        
  CD:
    runs-on: ubuntu-latest
    needs: [CI]
    steps:
      - name: Checkout do repositorio
        uses: actions/checkout@v4.1.1
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{secrets.AWS_ACCESS_KEY_ID}}
          aws-secret-access-key: ${{secrets.AWS_SECRET_ACCESS_KEY_ID}}
          aws-region: us-east-1
      - name: Configurar Kubeconfig
        run: aws eks update-kubeconfig --name eks-imersao --region us-east-1
      - name: Deploy to Kubernetes cluster
        uses: Azure/k8s-deploy@v4.9
        with:
          manifests: |
            ./kube-news/k8s/deployment.yaml
          images: |
            leolessa10/kube-news:${{ github.run_number }}
GCP
name: CI-CD

on:
  push:
    branches: ["main"]
  workflow_dispatch:

jobs:
  CI:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout do repositorio
        uses: actions/checkout@v4.1.1
      - name: Docker Login
        uses: docker/login-action@v3.0.0
        with:
          username: ${{secrets.DOCKERHUB_USER}}
          password: ${{secrets.DOCKERHUB_PWD}}
      - name: Build and push Docker images
        uses: docker/build-push-action@v5.0.0
        with:
          context: ./kube-news/src
          file: ./kube-news/src/Dockerfile
          push: true
          tags: |
            leolessa10/kube-news:${{ github.run_number }}
            leolessa10/kube-news:latest

  CD:
    runs-on: ubuntu-latest
    needs: [CI]
    steps:
      - name: Checkout do repositorio
        uses: actions/checkout@v4.1.1
      - name: Autenticar no GCP
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - name: Configurar gcloud
        run: gcloud config set project ${{ secrets.GCP_PROJECT_ID }}

      - name: Configurar Kubeconfig
        run: |
          gcloud container clusters get-credentials nome-do-cluster --zone us-central1-a

      - name: Deploy to GKE cluster
        uses: Azure/k8s-deploy@v4.9
        with:
          manifests: |
            ./kube-news/k8s/deployment.yaml
          images: |
            leolessa10/kube-news:${{ github.run_number }}
