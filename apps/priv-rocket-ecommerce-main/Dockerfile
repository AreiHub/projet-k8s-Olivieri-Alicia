FROM python:3.11.5-slim

# set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Bonne pratique : définir un dossier de travail
WORKDIR /app

COPY requirements.txt .

# install python dependencies (combiné pour réduire les layers)
RUN pip install --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

COPY . .

# Install node 18 and npm (Optimisé : on vide le cache apt à la fin pour alléger l'image)
RUN set -uex; \
    apt-get update; \
    apt-get install -y ca-certificates curl gnupg; \
    mkdir -p /etc/apt/keyrings; \
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
     | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg; \
    NODE_MAJOR=18; \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" \
     > /etc/apt/sources.list.d/nodesource.list; \
    apt-get update; \
    apt-get install nodejs -y; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/*

# Install modules and webpack
RUN npm i && npm run build

# Manage Assets & DB
RUN python manage.py collectstatic --no-input && \
    python manage.py makemigrations && \
    python manage.py migrate

# gunicorn
CMD ["gunicorn", "--config", "gunicorn-cfg.py", "core.wsgi"]