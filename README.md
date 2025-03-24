# Detecção de Objetos Cortantes com YOLO

## Objetivos
- Construir ou buscar um dataset contendo imagens de facas, tesouras e outros objetos cortantes em diferentes condições de ângulo e iluminação.
- Anotar o dataset para treinar o modelo supervisionado, incluindo imagens negativas (sem objetos perigosos) para reduzir falsos positivos.
- Treinar o modelo.
- Desenvolver um sistema de alertas (pode ser um e-mail).

## Estrutura do Projeto
O projeto está organizado da seguinte forma, algumas pastas só estarão disponiveis após execução:
```
.
├── custom_data/
│   ├── images/
│   └── labels/
├── datasets/
│   ├── data/
│   │   ├── train/
│   │   │   ├── images/
│   │   │   └── labels/
│   │   └── validation/
│   │       ├── images/
│   │       └── labels/
├── frames/
├── runs/
│   └── detect/
│       └── train/
│           └── weights/
├── videos/
├── data.yaml
├── Dockerfile
├── docker-compose.yml
└── execute.ipynb
```

## Configuração do Ambiente
### Verificação de GPU
Para verificar se a GPU está disponível, execute:
```bash
!nvidia-smi
```

### Instalação de Pacotes Necessários
Instale os pacotes necessários com os seguintes comandos:
```bash
!pip install numpy==2.1.1
!pip install tensorflow==2.9
!pip install --upgrade torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
!pip install ultralytics
```

## Preparação dos Dados
### Separação dos Dados em Treino e Teste
Utilize a função `train_val_split` para separar os dados:
```python
train_val_split("./custom_data/", 0.9)
```

### Criação do Arquivo `data.yaml`
Crie o arquivo `data.yaml` com as configurações do dataset:
```python
create_data_yaml('./custom_data/classes.txt', './data.yaml')
```

## Treinamento do Modelo
Treine o modelo YOLO com o seguinte comando:
```bash
!yolo detect train data=./data.yaml model=yolo11s.pt epochs=60 imgsz=640 device=0
```

## Validação do Modelo
Valide o modelo treinado com o comando:
```bash
!yolo detect predict model=runs/detect/train/weights/best.pt source=datasets/data/validation/images save=True conf=0.5
```

## Demonstração das Detecções
Para visualizar as detecções, utilize o seguinte código:
```python
import glob
from IPython.display import Image, display
for image_path in glob.glob(f'./runs/detect/predict/*.jpg')[:10]:
  display(Image(filename=image_path, height=400))
  print('\n')
```

## Envio de E-mail
Para enviar e-mails com as detecções, utilize a função `send_email`:
```python
async def send_email(subject, body, to_email, image_paths):
    # Implementação da função de envio de e-mail
```

## Análise de Objetos Cortantes
Para analisar objetos cortantes em imagens, pastas, vídeos ou câmera USB, utilize a função `detect_objects`:
```python
detect_objects(model_path, video_path, 0.45, "1920x1080", True)
```

## Docker
### Configuração do Docker
O projeto inclui um `docker-compose.yml` para configurar os serviços necessários:
```yaml
services:
  nginx:
    build: .
    image: heartexlabs/label-studio:latest
    restart: unless-stopped
    ports:
      - "8080:8085"
      - "8081:8086"
    depends_on:
      - app
    environment:
      - LABEL_STUDIO_HOST=${LABEL_STUDIO_HOST:-}
    volumes:
      - ./mydata:/label-studio/data:rw
      - ./deploy/nginx/certs:/certs:ro
    command: nginx

  app:
    stdin_open: true
    tty: true
    build: .
    image: heartexlabs/label-studio:latest
    restart: unless-stopped
    expose:
      - "8000"
    depends_on:
      - db
    environment:
      - DJANGO_DB=default
      - POSTGRE_NAME=postgres
      - POSTGRE_USER=postgres
      - POSTGRE_PASSWORD=
      - POSTGRE_PORT=5432
      - POSTGRE_HOST=db
      - LABEL_STUDIO_HOST=${LABEL_STUDIO_HOST:-}
      - JSON_LOG=1
    volumes:
      - ./mydata:/label-studio/data:rw
    command: label-studio-uwsgi

  db:
    image: pgautoupgrade/pgautoupgrade:13-alpine
    hostname: db
    restart: unless-stopped
    environment:
      - POSTGRES_HOST_AUTH_METHOD=trust
    volumes:
      - ${POSTGRES_DATA_DIR:-./postgres-data}:/var/lib/postgresql/data
      - ./deploy/pgsql/certs:/var/lib/postgresql/certs:ro
```

### Acessando o Site
Para acessar o site na máquina local, utilize `http://localhost:8080/`. Crie um projeto e selecione o setup de detecção de objetos para categorizar as imagens.

## Conclusão
Este projeto fornece uma solução completa para a detecção de objetos cortantes utilizando o modelo YOLO, incluindo a preparação dos dados, treinamento, validação, demonstração das detecções e envio de alertas por e-mail.