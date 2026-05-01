FROM python:3.11-slim

# Instalar dependências do sistema e Java (necessário para o Spark)
RUN apt-get update && \
    mkdir -p /usr/share/man/man1 && \
    apt-get install -y --no-install-recommends \
    default-jre-headless \
    curl \
    procps \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Configurar variáveis de ambiente do Java (ajustado para o caminho do default-jre)
ENV JAVA_HOME=/usr/lib/jvm/default-java
ENV PATH=$PATH:$JAVA_HOME/bin

# Instalar Poetry
ENV POETRY_VERSION=1.8.2
RUN curl -sSL https://install.python-poetry.org | python3 -
ENV PATH="/root/.local/bin:$PATH"

# Configurar diretório de trabalho
WORKDIR /app

# Copiar arquivos de dependências
COPY pyproject.toml poetry.lock* ./

# Instalar dependências via Poetry (sem criar ambiente virtual dentro do container)
RUN poetry config virtualenvs.create false \
    && poetry install --no-interaction --no-ansi

# Expor a porta do Jupyter Lab
EXPOSE 8888

# Comando para iniciar o Jupyter Lab
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", "--ServerApp.token=''", "--ServerApp.password=''", "--ServerApp.allow_origin='*'", "--ServerApp.disable_check_xsrf=True"]
