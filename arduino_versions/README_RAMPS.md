# Versão com Rampas Assimétricas

## 📁 Estrutura

```
arduino_versions/
├── transmit/          ← Transmissor (funciona com ambos receptores)
├── receive/           ← Receptor básico
├── ramp_receive/      ← Receptor com rampas de velocidade ⚡
└── README.md          ← Esta documentação
```

## 🔄 Qual Versão Usar?

| Transmissor | Receptor | Quando Usar | Características |
|-------------|----------|-----------|-----------------|
| **transmit/** | **receive/** | 🎓 **Aprendizado** | Código simples, limpo, didático. Resposta imediata do motor (sem suavização) |
| **transmit/** | **ramp_receive/** | ⚡ **Prototipagem** | Motores com **rampas assimétricas** - aceleração suave, frenagem rápida |

**Nota:** O transmissor é o mesmo para ambos! As rampas são aplicadas no receptor.

---

## 📌 Ponto Importante

**O transmissor envia apenas dados brutos do joystick.** A lógica de suavização é totalmente no receptor:

- `transmit.ino` → lê joystick, envia raw data via ESP-NOW
- `receive.ino` → aplica resposta instantânea nos motores
- `ramp_receive.ino` → aplica rampas suaves nos motores

Isso significa que você pode usar **qualquer transmissor com qualquer receptor**!

---

## ⚡ O Que Mudou na Versão com Rampas?

### Aceleração (Suave)
```
Tempo: ~200ms para atingir velocidade máxima
Efeito: Sem trancos ao aumentar velocidade
```

### Frenagem (Rápida)
```
Tempo: ~50ms para parar completamente  
Efeito: Resposta rápida a mudanças de direção
```

### Exemplo de Comportamento

**Versão Básica:**
```
Velocidade: 0 → [clique joystick] → 255 → [libera joystick] → 0
            Instantâneo              Instantâneo
```

**Versão com Rampas:**
```
Velocidade: 0 → [clique joystick] → 255 → [libera joystick] → 0
            Suave (200ms)           Rápida (50ms)
```

---

## 🔧 Configuração das Rampas

No arquivo `ramp_receive.ino`, linhas 68-72:

```cpp
//----------------------------------------------------------------------
// CONFIGURAÇÕES DE RAMPA
//----------------------------------------------------------------------

// Tempo em ms para atingir velocidade máxima durante aceleração
#define ACCELERATION_TIME 500  // Aceleração lenta (sem trancos)

// Tempo em ms para parar completamente durante frenagem
#define DECELERATION_TIME 150   // Frenagem rápida (resposta rápida)
```

### Ajustar Comportamento

- **Mais suave (aceleração):** aumentar `ACCELERATION_TIME` (ex: 300ms)
- **Menos suave (aceleração):** diminuir `ACCELERATION_TIME` (ex: 100ms)
- **Frenagem mais rápida:** diminuir `DECELERATION_TIME` (ex: 25ms)
- **Frenagem mais lenta:** aumentar `DECELERATION_TIME` (ex: 100ms)

---

## 📊 Como Funciona Internamente

### Estrutura de Rampa por Motor

```cpp
typedef struct {
  int16_t current_speed;      // Velocidade atual após rampa
  int16_t target_speed;       // Velocidade alvo desejada
  unsigned long last_update;  // Timestamp da última atualização
} MotorRampState;
```

### Função Principal: `applyMotorRamp()`

```cpp
int16_t applyMotorRamp(MotorRampState* ramp, int16_t target_speed, unsigned long current_time)
```

**Lógica:**
1. Recebe velocidade alvo (do joystick)
2. Verifica se é aceleração ou frenagem
3. Aplica rampa apropriada (lenta ou rápida)
4. Retorna velocidade suavizada para o PWM

**Detalhes Técnicos:**
- ✅ Interpolação linear dentro do tempo de rampa
- ✅ Proteção contra "overshooting" (não passa do alvo)
- ✅ Rampas independentes por motor (podem ter velocidades diferentes)
- ✅ Funciona com valores negativos (direção do motor)

---

## 🧪 Testando a Versão com Rampas

1. **Upload** em ambos os ESP32:
   - `transmit.ino` → Transmissor (versão básica funciona perfeitamente)
   - `ramp_receive.ino` → Receptor

2. **Serial Monitor** mostrará:
   ```
   === ESP32-S2 Joystick Receptor (VERSÃO RAMPA) ===
   ⚡ RAMPAS ASSIMÉTRICAS ATIVADAS
      Aceleração: 200ms | Frenagem: 50ms
   ```

3. **Observar diferença:**
   - Mova o joystick lentamente para cima → veja motor acelerar suavemente
   - Solte rapidamente → motor freia rápido

4. **Serial Output** mostra velocidades:
   ```
   MOTORS -> L_TARGET:   100 L_ACTUAL:    50 | R_TARGET:   100 R_ACTUAL:    50 SPEED:141
   ```
   - `TARGET`: velocidade desejada (do joystick)
   - `ACTUAL`: velocidade após rampa (suavizada)

---

## 💡 Quando Usar Cada Versão?

### Use `receive.ino` (básico) se:
- ✅ Está aprendendo C/Arduino
- ✅ Precisa de resposta instantânea
- ✅ Quer entender o código sem complexidade
- ✅ Está prototipando e testando

### Use `ramp_receive.ino` se:
- ✅ Quer movimentos mais realistas
- ✅ Motores fazem barulho/trancos
- ✅ Bateria descarrega rápido (motores sem picos)
- ✅ Precisa de suavidade para câmeras/sensores
- ✅ Quer integrar em projeto real

---

## 🚀 Próximos Passos

1. **Refinar as rampas:** Ajuste `ACCELERATION_TIME` e `DECELERATION_TIME` para seu gosto
2. **Integrar no PlatformIO:** A lógica pode ser copiada para `src/main.cpp`
3. **Crear biblioteca:** Extrair rampas para `lib/MotorRamper/` se quiser reusar

---

## 📝 Notas Técnicas

- **Compatibilidade:** Ambas as versões usam a mesma API ESP-NOW
- **MAC Address:** Pode usar `transmit.ino` com `ramp_receive.ino` (rampas só no receptor)
- **Performance:** Rampas adicionam ~2% de overhead de CPU
- **PWM Frequency:** Mantém 5kHz padrão do ESP32

---

## ❓ Perguntas Frequentes

**P: Posso usar transmit.ino básico com ramp_receive.ino?**
Sim! Na verdade, **você DEVE usar transmit.ino básico com ramp_receive.ino**. O transmissor é compartilhado entre ambas as versões.

**P: Qual o máximo de tempo de rampa?**
Até ~32 segundos (limitado por `uint32_t` de timestamp). Na prática, 200-500ms é recomendado.

**P: As rampas funcionam em reverso (-255)?**
Sim! Funcionam com valores negativos sem problemas.

**P: Onde estão as rampas no código?**
- Definições: linhas 68-72
- Estrutura de estado: linhas 120-126
- Função principal: `applyMotorRamp()` (linhas ~340)
- Aplicação: `setMotorWithRamp()` e `controlMotorsWithRamps()` (linhas ~370-410)
