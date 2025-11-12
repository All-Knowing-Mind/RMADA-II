# DEVICE CLIENT GUIDE — Stage 2

## 📱 Como Onboard um Novo Dispositivo

Este guia descreve o processo completo para registrar e conectar um novo dispositivo LoRa ao sistema RMADA.

### Visão Geral da Segurança

```
┌─────────────────────────────────────────────────────┐
│                RMADA Device Onboarding               │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. Device gera par Dilithium (crypto pós-quantum) │
│  2. Device assina seu ID com Dilithium             │
│  3. Device envia assinatura para servidor          │
│  4. Servidor verifica assinatura (nativo, sem deps)│
│  5. Servidor autoriza e envia config WireGuard     │
│  6. Device conecta via WireGuard à rede segura     │
│  7. Device envia telemetria contínua               │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 🔧 Pré-requisitos

### No Servidor

- Node.js 18+ (ou Docker)
- Servidor RMADA rodando (`npm start` ou `docker-compose up`)
- Binários Dilithium compilados (`npm run build:dilithium-all`)

### No Dispositivo (Client)

- bash shell
- curl
- Acesso ao repositório RMADA (scripts)
- Rust + Cargo (SE quiser compilar localmente; senão binários pré-compilados)

---

## 📋 Passo 1: Preparar Servidor

### 1.1 Start Server (Local)

```bash
# Instalar deps
npm install

# Build Dilithium
npm run build:dilithium-all

# Start Node.js + WebSocket
npm start

# Deve imprimir:
# Server listening on :8080
# WebSocket listening on :9000
```

### 1.2 Start Server (Docker)

```bash
# Build
docker-compose build

# Run
docker-compose up -d

# Verificar
docker ps | grep rmada
```

### 1.3 Registrar Owner

```bash
# No seu computador ou em outro terminal
SERVER="http://localhost:8080"

curl -X POST $SERVER/api/register-owner \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "SecurePassword123!",
    "ownerCode": "OWNER-RMADA-001"
  }'

# Resposta esperada:
# {
#   "message": "Owner registered successfully",
#   "token": "abc123defghijklmnopqrst...",
#   "role": "owner"
# }

# Copie o token (use depois)
TOKEN="abc123defghijklmnopqrst..."
```

---

## 📱 Passo 2: Client Device (Simulado)

### 2.1 Preparar Cliente

```bash
# Clone ou copie o repositório para o "dispositivo"
# (neste exemplo, vamos rodar no mesmo host em outro terminal)

cd /tmp/device-simulation-$RANDOM
cp -r /path/to/rmada ./
cd rmada

# Ou baixe apenas os scripts:
curl -O https://raw.githubusercontent.com/seu-repo/device-client-example.sh
curl -O https://raw.githubusercontent.com/seu-repo/generate_dilithium_keys.sh
```

### 2.2 Executar Cliente

```bash
# Variáveis
SERVER="http://localhost:8080"
DEVICE_ID="DEVICE-SENSOR-001"
TOKEN="abc123defghijklmnopqrst..."

# Rodar cliente
bash device-client-example.sh "$SERVER" "$DEVICE_ID" "$TOKEN"

# Saída esperada:
# ================================================
# RMADA Device Client Example
# ================================================
# 
# Server:    http://localhost:8080
# Device ID: DEVICE-SENSOR-001
# Token:     abc123de...
# 
# 1️⃣  Setting up WireGuard keys...
#    ✓ WireGuard public key: iN8/aBcD...
# 
# 2️⃣  Setting up Dilithium keys...
#    ✓ Keys generated successfully!
# 
# 3️⃣  Reading Dilithium keys...
#    ✓ Public key length: 3104 hex chars
# 
# ... (continua)
```

### 2.3 O que Acontece no Background

```
Step 1: WireGuard Setup
├─ wg genkey > device.private
├─ cat device.private | wg pubkey > device.public
└─ Store both in device-keys-$DEVICE_ID/

Step 2: Dilithium Keygen
├─ Call: ./dilithium_keygen ./device-keys-$DEVICE_ID
├─ Generates: dilithium_public.key, dilithium_secret.key
└─ Keys are LOCAL (secret key never leaves device)

Step 3: Sign Device ID
├─ Message: "DEVICE-SENSOR-001" (utf8)
├─ Call: ./sign ./dilithium_secret.key ./message.txt > sig.bin
├─ Converts sig.bin to base64
└─ Result: assinatura de 8658 chars

Step 4: Device Onboarding
├─ POST /api/device-onboard
├─ Data: { deviceId, dilithium_pubkey (hex), dilithium_signature (b64), wg_pubkey }
└─ Server: verifica assinatura com dilithium_verify

Step 5: Retrieve WireGuard Config
├─ GET /api/get-wg-config/DEVICE-SENSOR-001
├─ Response: config WireGuard pronto para usar
└─ Client: salva em wg-config-$DEVICE_ID.conf

Step 6: Activate WireGuard (opcional)
├─ sudo wg-quick up ./wg-config-$DEVICE_ID.conf
└─ Device conecta à VPN (se tiver permissões)

Step 7: Send Telemetry (loop)
├─ Generate random sensor values (0-100)
├─ POST /api/telemetry { deviceId, value, timestamp }
├─ Repeat 5x com delay de 2 segundos
└─ Frontend visualiza em tempo real
```

---

## 🔐 Segurança: O que Está Protegido

### ✅ Protected

| Aspecto | Mecanismo |
|---------|-----------|
| Device authentication | Dilithium signature (post-quantum) |
| VPN keys | WireGuard (ephemeral) |
| Server verification | Native pqc_dilithium (nenhuma dep externa) |
| Telemetry transport | HTTPS + WireGuard (Stage 3) |

### ⚠️ Assume

- Servidor HTTPS (Stage 3 adiciona)
- Device local já "trusted" (já têm token owner)
- WireGuard nat traversal (precisa setup em firewall)

---

## 🎬 Exemplo Completo End-to-End

### Setup (Terminal 1 — Servidor)

```bash
$ cd /home/ubuntu/rmada
$ npm install
$ npm run build:dilithium-all
$ npm start

# Output:
# Server listening on :8080
# WebSocket listening on :9000
```

### Setup (Terminal 2 — Owner Registration)

```bash
$ curl -X POST http://localhost:8080/api/register-owner \
  -H "Content-Type: application/json" \
  -d '{
    "username":"owner1",
    "password":"test123",
    "ownerCode":"OWNER-001"
  }'

# Response:
# {"message":"Owner registered successfully","token":"eyJhbGc...","role":"owner"}

$ export OWNER_TOKEN="eyJhbGc..."
```

### Setup (Terminal 3 — Device Client)

```bash
$ cd /tmp/test-device
$ cp /home/ubuntu/rmada/device-client-example.sh .
$ cp /home/ubuntu/rmada/generate_dilithium_keys.sh .

$ bash device-client-example.sh http://localhost:8080 LORA-001 "$OWNER_TOKEN"

# Output completo (veja acima em 2.2)
```

### Verificar (Terminal 2 — API)

```bash
# Listar devices
$ curl http://localhost:8080/api/whoami \
  -H "Authorization: Bearer $OWNER_TOKEN" | jq .

# Response:
# {
#   "username": "owner1",
#   "role": "owner",
#   "devices": ["LORA-001"]
# }

# Ver config WireGuard do device
$ curl http://localhost:8080/api/get-wg-config/LORA-001 \
  -H "Authorization: Bearer $OWNER_TOKEN" | jq .

# Response:
# {
#   "deviceId": "LORA-001",
#   "config": "# WireGuard config...",
#   "device_ip": "10.0.0.2/32"
# }
```

### Dashboard (Terminal ou Browser)

```bash
# Abra
http://localhost:8080/Operação.html

# Deve mostrar:
# - Device LORA-001 conectado
# - 5 telemetrias chegando (gráfico animado)
# - Status do WireGuard
```

---

## 🚨 Troubleshooting

### Erro: "dilithium_verify: command not found"

**Causa**: Binário Dilithium não compilado ou não em PATH

**Fix**:
```bash
npm run build:dilithium-all
export PATH="/path/to/rmada/meu_projeto_dilithium/target/release:$PATH"
```

### Erro: "Dilithium verification failed"

**Causa**: Chaves não combinam (public/secret pair mismatch)

**Fix**:
```bash
# Certifique que secret key usado para assinar é do mesmo par que public key
bash generate_dilithium_keys.sh ./test-keys

# Teste manualmente
echo -n "TEST-MESSAGE" > msg.txt
./meu_projeto_dilithium/target/release/sign ./test-keys/dilithium_secret.key msg.txt > sig.bin
./meu_projeto_dilithium/target/release/dilithium_verify \
  ./test-keys/dilithium_public.key msg.txt sig.bin

# Deve retornar exit code 0
echo "Exit code: $?"
```

### Erro: "WireGuard keys generation failed"

**Causa**: `wg` tool não instalado

**Fix**:
```bash
# Debian/Ubuntu
sudo apt-get install wireguard-tools

# Arch
sudo pacman -S wireguard-tools

# macOS
brew install wireguard-tools
```

### Device pode assinar mas não conecta VPN

**Possibilidades**:
1. Firewall bloqueando UDP 51820
2. WireGuard não instalado no device
3. Config IP conflita com rede local

**Fix**:
```bash
# Verifique VPN config
cat wg-config-$DEVICE_ID.conf

# Teste manualmente (se tiver sudo)
sudo wg-quick up ./wg-config-$DEVICE_ID.conf
sudo wg show

# Debug
wg show dump
```

---

## 📊 Monitorar Telemetria

### Via API

```bash
# Último valor de um device
curl http://localhost:8080/api/telemetry?deviceId=LORA-001 \
  -H "Authorization: Bearer $TOKEN" | jq .

# Resposta:
# {
#   "deviceId": "LORA-001",
#   "lastValue": 42,
#   "timestamp": "2025-11-11T15:30:45.123Z",
#   "values": [55, 68, 42, 71, 38]  // últimos 5
# }
```

### Via WebSocket

```javascript
// No browser console
const ws = new WebSocket('ws://localhost:9000');
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log(`Device ${data.deviceId}: ${data.value}`);
};
ws.send(JSON.stringify({ subscribe: 'telemetry', deviceId: 'LORA-001' }));
```

### Via Dashboard HTML

Abra `Operação.html` no navegador — mostra gráficos em tempo real.

---

## 🔄 Multiple Devices

```bash
# Device 1
bash device-client-example.sh http://localhost:8080 LORA-001 "$TOKEN" &

# Device 2
bash device-client-example.sh http://localhost:8080 LORA-002 "$TOKEN" &

# Device 3
bash device-client-example.sh http://localhost:8080 LORA-003 "$TOKEN" &

# Todos enviam telemetria → Dashboard visualiza 3 gráficos
```

---

## 🎯 Next Steps

1. **Stage 3**: Adicione HTTPS (TLS com Dilithium certs) + Lightway
2. **Production**: Deploy em cloud (AWS/DigitalOcean/Azure) com Lightway gateway
3. **Mobile**: App React Native ou Progressive Web App
4. **Persistence**: SQLite para histórico de telemetria

---

**Status**: Stage 2 Device Client Ready  
**Data**: 2025-11-11
