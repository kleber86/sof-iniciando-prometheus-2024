# Instalando Prometheus

Baixando dentro do docker ou WSL uma imagem ubuntu

Criando container: `docker run -it --name PROMETHEUS ubuntu bash`

Criando usuario no ubuntu: `useradd --no-create-home --shell /bin/false prometheus`

Criando diretorios:
`mkdir /etc/prometheus`
`mkdir /var/lib/prometheus`

Dando acesso ao usuario e grupo:
`chown -R prometheus:prometheus /etc/prometheus`
`chown -R prometheus:prometheus /var/lib/prometheus`

Baixar o Prometheus: `https://github.com/prometheus/prometheus/releases/download/v2.51.0/prometheus-2.51.0.linux-amd64.tar.gz`

Pegar o shar256 e comparar: `sha256sum prometheus-2.51.0.linux-amd64.tar.gz`

Descompactar na home o Prometheus: `tar xvzf prometheus-2.51.0.linux-amd64.tar.gzz`

Acessar o diretório: `cd prometheus-2.51.0.linux-amd64/`

Copiar dois diretórios:
`cp prometheus /usr/local/bin/`
`cp promtool /usr/local/bin/`

Permissões:
`chown -R prometheus:prometheus /usr/local/bin/prometheus`
`chown -R prometheus:prometheus /usr/local/bin/promtool`

Copiar dois diretórios:
`cp -r consoles /etc/prometheus/`
`cp -r console_libraries /etc/prometheus/`

Acessar o diretório e veririficar os dois criados: `cd /etc/prometheus/`
consoles
console_libraries

Permissões:
`chown -R prometheus:prometheus /etc/prometheus/consoles`
`chown -R prometheus:prometheus /etc/prometheus/console_libraries`

Criar arquivo de inicialização e adicionar os dados abaixo: `vim /etc/systemd/system/prometheus.service`
```
[Unit]
Description=Prometheus Server
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Criar o arquivo prometheus.yml com os dados abaixo: `vim /etc/prometheus/prometheus.yml`
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```

Permissões:
`chown -R prometheus:prometheus /etc/prometheus/prometheus.yml`

Iniciar o serviço do Prometheus: `systemctl start prometheus`

Consultar o status do serviço: `systemctl status prometheus`

# Protegendo o Prometheus

Instalando um proxy-reverso: `apt-get install nginx -y`

Armazenar senha e criptografada: `apt-get install apache2-utils -y`

Cria um arquivo que armazena um usuario e senha: `htpasswd -c /etc/nginx/.htpasswd son`

Local desse arquivo: `cd /etc/nginx/.htpasswd`

Criar um v-host prometheus: `/etc/nginx/sites-available`

Criar a copia do default: `cp default prometheus`

Remover o default dos dois diretorios: `rm -rf default`
```
/etc/nginx/sites-available
/etc/nginx/sites-enabled
```

Configurar o arquivo para solicitar autenticação: `vim /etc/nginx/sites-available/prometheus`
```
location / {
  # deny all;
  auth_basic "Prometheus Server Auth";
  auth_basic_user_file /etc/nginx/.htpasswd;
  proxy_pass http://192.168.237.113:9090;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
}
```

Criar um link desse arquivo: `ln -s /etc/nginx/sites-available/prometheus  /etc/nginx/sites-enabled/prometheus`

Verificar a sintaxe do nginx: `nginx -t`

Reiniciar o serviço: `systemctl restart nginx`

Consultar o status do serviço: `systemctl status nginx`

# Instalando Node Exporter

Criar novo usuario: `sudo useradd --no-create-home --shell /bin/false node_exporter`

Baixando o node exporter: 
`wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz`

Descompactando o Node Exporter: `sudo tar xvzf node_exporter-1.7.0.linux-amd64.tar.gz`

Copiando o binário: `sudo cp node_exporter /usr/local/bin/`

Permissões: `sudo chown -R node_exporter:node_exporter /usr/local/bin/node_exporter`

Criando o arquivo de service: `sudo vim /etc/systemd/system/node_exporter.service`
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter 

[Install]
WantedBy=multi-user.target
```

Iniciar o serviço: `sudo systemctl start node_exporter`

Consultar o serviço: `sudo systemctl status node_exporter`

Endereço das metricas: `curl localhost:9100/metrics`

# Configurando Prometheus para varrer metrica

Editar o arquivo prometheus.yml, adicionar os dados abaixo: `sudo vim /etc/prometheus/prometheus.yml`
```
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```

Reiniciar o servico: `sudo systemctl restart prometheus`

Consultar o status: `sudo systemctl status prometheus`

# Instalando Grafana

`sudo apt-get install -y apt-transport-https software-properties-common wget`

`sudo mkdir -p /etc/apt/keyrings/ wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null`

`echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list`

`echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list`

`echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list`

`sudo apt-get install grafana`

Iniciar o servico: `sudo systemctl start grafana-server`

Consultar o status: `sudo systemctl status grafana-server`

# Integrando Prometheus e Grafana

Usuario e senha padrão: `admin` e `admin`

Adicionar um data source:
```
prometheus
Connection: http://localhost:9090
Save & test
```