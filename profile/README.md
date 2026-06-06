# Kakatua013

Sistema de bicicletário inteligente com controle de vagas via sensor ultrassônico e gerenciamento web.

---

## Sobre o projeto

O **Kakatua Bikes** é um sistema integrado de bicicletário que combina hardware e software para controlar o acesso e a disponibilidade de vagas para bicicletas. Um sensor ultrassônico instalado em cada vaga detecta a presença de uma bicicleta e se comunica com um servidor via HTTP. O servidor processa as informações, atualiza o banco de dados e serve uma interface web onde o usuário pode visualizar o status da estação, solicitar liberação da tranca e autenticar-se no sistema. O administrador tem acesso adicional para gerenciar credenciais e visualizar logs de acesso.

---

## Casos de uso

O diagrama abaixo mostra os atores do sistema e o que cada um pode fazer:

![Diagrama de Caso de Uso](https://raw.githubusercontent.com/Kakatua013/KakatuaUMLs/main/Kakatua%20Diagrama%20de%20Caso%20de%20Uso.png)

| Ator | Ações disponíveis |
|------|-------------------|
| Usuário | Solicitar liberação da tranca, visualizar status da estação, autenticar-se no sistema |
| Administrador | Gerenciar credenciais, visualizar logs de acesso, autenticar-se no sistema |

---

## Arquitetura de componentes

O diagrama de componentes mostra como os módulos do sistema se relacionam entre si:

![Diagrama de Componentes](https://raw.githubusercontent.com/Kakatua013/KakatuaUMLs/main/Kakatua%20Diagrama%20de%20Componentes.png)

O sistema é dividido em quatro componentes principais:

- **Proxy (Nginx)** — recebe todas as requisições externas e as redireciona para o Django. É o ponto de entrada do sistema.
- **Servidor (Django)** — núcleo da aplicação. Recebe requisições da interface web e sinais do ESP32, consulta e escreve no banco de dados, e envia sinais de volta para o hardware.
- **Interface Web** — permite ao usuário visualizar o estado da estação, fazer login, cadastro e consultar o banco de dados.
- **Hardware ESP32** — envia sinais de disponibilidade de vaga para o servidor e aciona a tranca física conforme resposta recebida.
- **Estação Física** — componente de hardware que executa o bloqueio/desbloqueio da tranca e verifica a disponibilidade via sensor.
- **Banco de Dados (MySQL)** — responsável pelo armazenamento de dados e hashing de senhas.

---

## Diagrama de implantação

O diagrama de implantação mostra como os componentes estão distribuídos fisicamente e quais protocolos de comunicação são utilizados:

![Diagrama de Implantação](https://raw.githubusercontent.com/Kakatua013/KakatuaUMLs/main/Kakatua%20Diagrama%20de%20Implanta%C3%A7%C3%A3o.png)

- **Celular / Computador** — o usuário acessa o sistema pelo navegador web usando o protocolo HTTP/HTTPS.
- **Estação de Bicicletas** — composta pelo Hardware ESP32, sensor ultrassônico e tranca eletrônica. Comunica-se com o servidor via protocolo HTTP.
- **Servidor** — hospeda a aplicação Django, o Proxy (Nginx) e o banco de dados MySQL. Recebe requisições de ambos os clientes acima.

---

## Repositórios

### [KakatuaSoftware](https://github.com/Kakatua013/KakatuaSoftware)
Backend e frontend web do sistema, desenvolvido em **Django** com **Django REST Framework**, servido via **Nginx** como proxy reverso.

**O que faz:**
- Recebe requisições da interface web (login, cadastro, visualização de status)
- Recebe sinais do ESP32 com o estado de cada vaga
- Consulta e escreve no banco de dados MySQL
- Envia sinais de resposta de volta ao ESP32 (abrir/fechar tranca)
- Autentica requisições dos dispositivos via API Key (header `X-API-Key`)

**Principais tecnologias:**
- Python / Django
- Django REST Framework
- MySQL (banco de dados)
- Nginx (proxy reverso)
- HTML/CSS (templates do site)

**Como rodar localmente:**
```bash
# Instale as dependências
pip install django
pip install djangorestframework
pip install markdown
pip install django-filter

# Ou via arquivo de requirements
pip install -r requirements.txt

# Suba o servidor
python manage.py runserver
```

**Endpoints da API:**

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/api/sensor/estacao/` | Atualiza o status geral da estação |
| POST | `/api/sensor/acesso/` | Atualiza o estado de uma tranca específica |

---

### [Kakatua_ESP32](https://github.com/Kakatua013/Kakatua_ESP32)
Firmware do microcontrolador **ESP32** responsável pela detecção de bicicletas e comunicação com o servidor.

**O que faz:**
- Conecta à rede Wi-Fi e mantém comunicação HTTP com o servidor
- Aciona o sensor ultrassônico (HC-SR04) a cada 100ms para medir a distância à frente da vaga
- Se a distância for ≤ 10 cm, considera a vaga ocupada e envia sinal para fechar a tranca e bloquear a estação
- Se a vaga estiver livre, envia sinal para abrir a tranca e desbloquear a estação
- Controla um LED indicativo (pino 21) que acende quando a vaga está ocupada

**Fluxo de funcionamento:**
```
Loop a cada 100ms
  ├── Dispara pulso ultrassônico (pino Trig: 13)
  ├── Lê tempo de retorno (pino Echo: 14)
  ├── Calcula distância em cm
  ├── distância ≤ 10cm → vaga ocupada
  │     ├── Acende LED
  │     └── POST /api/sensor/acesso/  → tranca: "fechada"
  │         POST /api/sensor/estacao/ → bloqueada: true
  └── distância > 10cm → vaga livre
        ├── Apaga LED
        └── POST /api/sensor/acesso/  → tranca: "aberta"
            POST /api/sensor/estacao/ → bloqueada: false
```

**Bibliotecas necessárias (Arduino IDE):**
- `WiFi.h` — conexão com rede Wi-Fi
- `HTTPClient.h` — requisições HTTP para a API
- `ArduinoJson.h` — serialização do payload JSON

**Configuração antes de gravar:**
```cpp
const char* ssid     = "NOME_DA_REDE";
const char* password = "SENHA_DA_REDE";
const char* apiKey   = "SUA_API_KEY";
```

---

### [KakatuaUMLs](https://github.com/Kakatua013/KakatuaUMLs)
Diagramas UML do projeto: caso de uso, componentes e implantação. Gerados com **ModelIO** e usados como documentação de análise e projeto do sistema.

---

## Autores

| Nome | GitHub |
|------|--------|
| Guilherme Souza Santos | [@guilherme-968](https://github.com/guilherme-968) |
| Davi Sousa Alves | [@Dabiito](https://github.com/Dabiito) |
| Gustavo Santiago de Almeida | [@GustavoLagartixa](https://github.com/GustavoLagartixa) |
| Edwin Sasaki | [@HideonHills](https://github.com/HideonHills) |
| Raul Santos | [@rauS2](https://github.com/rauS2) |
