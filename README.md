## Sobre
Tutorial sobre como integrar uma aplicação Maven no GitHub com o Docker Hub.

## Requisitos
- Ter um projeto Maven no GitHub
- Ter uma conta no Docker Hub

## Passo a passo

### Configurando dados de acesso ao Docker Hub
- Logar no GitHub
- Acessar o repositório do projeto
- Clicar na aba Settings -> Secrets and variables -> Actions
- Clicar no botão 'New Repository Secret'
- Preencher os campos:
```
Name: <nome-usuario-no-docker-hub>
Secret: <senha-no-docker-hub>
```

### Configurando um GitHub Actions
- Clicar na aba 'Actions'
- Localizar a opção 'Java with Maven' e clicar no botão 'Configure'
- Alterar o nome 'maven.yml' para <algum-nome-de-sua-preferencia>
- Adicione o seguinte conteudo
```
name: <algum-nome-de-sua-preferencia>

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout applicaion
      uses: actions/checkout@v3

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Build application
      run: mvn clean package

    - name: Build application image
      run: docker compose build

    - name: Set tag of image to Docker Hub
      run: echo "IMAGE_TAG=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)" >> $GITHUB_ENV

    - name: Login to Docker Hub
      uses: docker/login-action@v2.0.0
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Push application image to Docker Hub
      run: |
        docker tag ${{ secrets.DOCKER_USERNAME }}/<nome-projeto>:latest ${{ secrets.DOCKER_USERNAME }}/<nome-projeto>:$IMAGE_TAG
        docker push ${{ secrets.DOCKER_USERNAME }}/<nome-projeto>:$IMAGE_TAG
        docker push ${{ secrets.DOCKER_USERNAME }}/<nome-projeto>:latest
```
- Clicar no botão 'Commit changes'
- Adicionar uma mensagem no campo 'Commit message' e clicar no botão 'Commit changes'
- Clicar na aba 'Actions', em seguida no workflow <nome-utilizado-no-item-name>
- Clicar no item 'Primeiro commit', depois no botão 'Build' e acompanhar o processo

### Verificando
Logar no Docker Hub e verificar se a imagem do projeto foi gerada corretamente.

## Licença
Este projeto está sob licença do MIT. Para mais detalhes, ver o arquivo LICENSE.
