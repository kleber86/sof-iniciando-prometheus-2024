# Instalando Prometheus

Baixando dentro do docker ou WSL uma imagem ubuntu

Criando container: `docker run -it --name PROMETHEUS ubuntu bash`

Criando usuario no ubuntu: `useradd --no-create-home --shell /bin/false prometheus`

Criando diretorios:
mkdir /etc/prometheus
mkdir /var/lib/prometheus

Dando acesso ao usuario e grupo:
chown -R prometheus:prometheus /etc/prometheus
chown -R prometheus:prometheus /var/lib/prometheus

Baixar o Prometheus: https://github.com/prometheus/prometheus/releases/download/v2.51.0/prometheus-2.51.0.linux-amd64.tar.gz

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