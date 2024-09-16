## How to use

### 0. Set Cohere API Key
```
export COHERE_API_KEY=xxxx
```

### 1. Run nginx for testing AI Respons Transformer Plugin
```
PROXY=localhost:8000
docker run --name nginx --network=kong-net -d nginx
```

### 2. Run Kong Gateway
```
docker network create kong-net
```
```
docker run -d --name kong-database \
  --network=kong-net \
  -p 5432:5432 \
  -e "POSTGRES_USER=kong" \
  -e "POSTGRES_DB=kong" \
  -e "POSTGRES_PASSWORD=kongpass" \
  postgres:13
```
```
docker run --rm --network=kong-net \
 -e "KONG_DATABASE=postgres" \
 -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_PASSWORD=kongpass" \
 -e "KONG_PASSWORD=test" \
kong/kong-gateway:3.7.1.2 kong migrations bootstrap
```
```
docker run -d --name kong-gateway \
 --network=kong-net \
 -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_USER=kong" -e "KONG_PG_PASSWORD=kongpass" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
 -e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
 -e "KONG_COHERE_AUTH_HEADER=Bearer $COHERE_API_KEY" \
 -p 8000:8000 \
 -p 8001:8001 \
 -p 8002:8002 \
 kong/kong-gateway:3.7.1.2
```

### 3. Sync Deck YAML to Gateway
```
deck gateway sync deck/
```
