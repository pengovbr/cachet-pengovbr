# 📘 Deploy do Cachet

Este documento contém o roteiro completo para subir o projeto [Cachet](https://github.com/pengovbr/cachet-pengovbr) em um servidor Linux com NGINX e PHP 8.2.

---

## 🚀 Roteiro para Implantação do Cachet

### 1. Atualizar o servidor e instalar dependências

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx php8.2 php8.2-fpm php8.2-cli php8.2-gd php8.2-mysql php8.2-pgsql php8.2-curl php8.2-mbstring php8.2-xml php8.2-zip git composer redis-server
```

---

### 2. Criar e configurar o banco de dados

Acesse o MySQL:

```bash
mysql -u root -p
```

Dentro do MySQL:

```sql
CREATE DATABASE cachet_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'cachet_user'@'localhost' IDENTIFIED BY 'senha_segura';
GRANT ALL PRIVILEGES ON cachet_db.* TO 'cachet_user'@'localhost';
FLUSH PRIVILEGES;
```

---

### 3. Clonar o projeto e instalar dependências

```bash
sudo git clone -b 3.x https://github.com/cachethq/cachet.git /var/www/cachet
cd /var/www/cachet
composer install --no-dev -o
composer update cachethq/core
```

---

### 4. Configurar o ambiente

```bash
cp .env.example .env
php artisan key:generate
php artisan vendor:publish --tag=cachet
```

Edite o arquivo `.env` com as configurações adequadas:

```env
APP_URL=https://cachet.gestaopen.gov.br
DB_DATABASE=cachet_db
DB_USERNAME=cachet_user
DB_PASSWORD=senha_segura
```

---

### 5. Ajustar permissões

```bash
sudo chown -R www-data:www-data /var/www/cachet
sudo chmod -R 775 storage bootstrap/cache
```

---

### 6. Rodar as migrações e cria o primeiro usuário

```bash
php artisan migrate --force
php artisan cachet:make:user
```

---

### 7. Habilitar o cron job

```bash
* * * * * php /path/to/cachet/artisan schedule:run >> /dev/null 2>&1
```

---

### 8. Configurar o NGINX

Crie o arquivo `/etc/nginx/sites-available/cachet` com o seguinte conteúdo:

```nginx
server {
    listen 80;
    server_name cachet.gestaopen.gov.br;

    root /var/www/cachet/public;
    index index.php index.html;

    location / {
        try_files $uri /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Ative a configuração:

```bash
sudo ln -s /etc/nginx/sites-available/cachet /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### ✅ Finalização

Acesse `http://cachet.gestaopen.gov.br` no navegador e finalize a instalação pela interface web do Cachet.
