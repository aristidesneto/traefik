# Traefik 2 - Wordpress, Mysql e Portainer

## Docker e Docker-compose

Necessário ter o docker e docker-compose instalado em seu servidor:

```
# Docker
curl -fsSL https://get.docker.com | sh

# Docker-compose
curl -L https://github.com/docker/compose/releases/download/1.25.3/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

## Subindo sua aplicação Wordpress

Clone o projeto:

```
git clone https://github.com/aristidesneto/traefik.git && cd traefik-wordpress
```

Crie o arquivo de configuração `.env` a partir do `.env.example`

```
cp .env.example .env
```

Abre o arquivo `.env` e informe os dados solicitados:

```
# Senha Wordpress e MySQL
WORDPRESS_DB_PASSWORD=password
MYSQL_ROOT_PASSWORD=password

# Domínio da aplicação 
HOST_WP=myapp.your-domain.com

# Portainer
HOST_PORTAINER=portainer.your-domain.com
```

Crie o arquivo de configuração dinâmico do traefik:

```
cp config/traefik_dynamic.example.toml config/traefik_dynamic.toml
```

Para proteger a rota do Dashboard do Traefik, iremos utilizar o utilitário `htpasswd` do Apache. É necessário instalarmos o pacote `utils`. Instale da seguinte maneira:

```
apt install apache2-utils
```

Gerando uma senha para o usuário `admin`:

```
htpasswd -nb admin my_secret_password
```

O resultado será algo como:

```
admin:$apr1$t8.q6q7g$V0GH0FOVeTd796LWLmQzX1
```

Edite o arquivo `config/traefik_dynamic.toml`:

```
[http.middlewares.simpleAuth.basicAuth]
  users = [
    # Cole aqui a senha criada
    "admin:$apr1$t8.q6q7g$V0GH0FOVeTd796LWLmQzX1" 
  ]

[http.routers.api]
  # Altere seu domínio aqui
  rule = "Host(`traefik.your-domain.com`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"
```

Altere o arquivo de configuração `config/traefik.toml` e adicione seu e-mail para que seja utilizado pelo Let's Encrypt:

```
...
[certificatesResolvers.lets-encrypt.acme]
  email = "email@your-domain.com"
  storage = "acme.json"
...
```

Crie uma rede no docker para os containers:

```
docker network create traefik
```

Crie o arquivo `acme.json` dentro do diretório `letsencrypt`. Esse arquivo é responsável em armazenar os certificados SSL criados para aplicacão.

```
touch letsencrypt/acme.json
chmod 600 letsencrypt/acme.json
```

Por fim, execute `docker-compose up -d` para que os containers sejam criados. O Traefik irá gerenciar os containers que foram definidos no arquivo `docker-composer.yml` e automaticamente irá criar os certificados SSL para cada domínio informado.

## Acesso aos containers

Foram criados 4 containers docker e 3 deles estão expostos para acesso externo:

- Traefik: traefik.your-domain.com (informe o usuário e senha gerado através do utilitário utils do Apache)
- Portainer: portainer.your-domain.com
- Wordpress: myapp.your-domain.com
- Mysql: esse container não está exposto para acesso externo, sendo acessível somente para o container do wordpress

