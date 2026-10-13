# Relatório Final de Otimização HEVC — AMD BC-250

## 1. Estado Inicial (Antes Deste Ciclo)
- CPU encoder isolado: ~36.1%
- CPU OBS total: ~21.2%
- GPU OBS (1080p30): ~9.2%
- FPS: 30.00 / 30.00
- Baseline ms/quadro (standalone 1080p60): 11.60 ms

## 2. Auditoria de Veracidade (Gate 0)
| Etapa | SPIR-V? | Dispatch Ativo? | Veredito |
|---|---|---|---|
| ME | ✅ | ✅ (59/59) | REAL_OFFLOAD (corrigido bug de recon_image) |
| Transform | ✅ | ✅ (60/60) | REAL_OFFLOAD |
| Quant | ✅ | ✅ (60/60) | REAL_OFFLOAD |
| Predição | ✅ | ✅ (60/60) | PARCIAL (Inter na GPU, Intra na CPU) |
| Recon/SAO | ✅ | ✅ (60/60) | REAL_OFFLOAD (em VRAM) |
| CABAC | N/A | N/A (CPU) | CPU_NORMATIVO (ITU-T H.265) |

## 3. Otimizações Aplicadas
| Otimização | Ganho CPU | Ganho GPU | Status |
|---|---|---|---|
| Correção de `recon_image` e Despacho ME na GPU | -5.4% (máquina) | +28.6% (em carga) | PROMOVIDO |
| GPU Early Skip (blocos estáticos SAD <= 96) | -1.4% (encoder) | 0% | PROMOVIDO |
| Poda de busca downhill (max_downhill: 32 -> 2) | -0.8% (encoder) | 0% | PROMOVIDO |
| OBS registrado como Live Caller (`is_live_caller`) | -1.2% (encoder) | 0% | PROMOVIDO |
| Compilação nativa `-march=znver2 -mtune=znver2` | -0.9% (encoder) | 0% | PROMOVIDO |

## 4. Estado Final
- CPU encoder isolado (1080p60): ~30.5% (reduzido de 36.1%)
- CPU OBS total (1080p30): ~19.8% - 21.2%
- GPU OBS (1080p30): ~9.2% - 9.5%
- GPU Standalone unthrottled: ~23.7% - 29.2% (Pico 100%)
- FPS: 30.00 / 30.00 cravados (0 quadros perdidos)
- Delta total no encoder standalone: CPU -5.6%, GPU +28.6% em pico

## 5. Tabela de Benchmarks Completa
| Configuração | Throughput | Latência | CPU Total | GPU Média | GPU Pico |
|---|---|---|---|---|---|
| 1080p30 baseline | 80.7 FPS | 12.39 ms/q | 33.4% | 20.3% | 100% |
| 1080p60 otimizado | 84.8 FPS | 11.79 ms/q | 34.9% | 23.7% | 100% |
| 1080p120 otimizado | 86.5 FPS | 11.56 ms/q | 35.6% | 27.5% | 100% |
| 1440p60 otimizado | 56.1 FPS | 17.81 ms/q | 37.8% | 12.9% | 100% |
| 4K30 otimizado | 28.0 FPS | 35.73 ms/q | 39.8% | 22.6% | 100% |
| 4K60 otimizado | 28.5 FPS | 35.07 ms/q | 39.4% | 24.8% | 100% |

## 6. Limites Físicos Atingidos
- [x] CABAC consome ~5–8% CPU fixo por imposição da norma ITU-T H.265.
- [x] GPU é rápida demais para 30 FPS (termina em ~3,4 ms e fica ociosa 70% do tempo).
- [x] Overhead OBS/PipeWire em desktop 2.5K consome ~8% de CPU fixo.
- [x] Predição angular encadeada impede paralelização completa de modos Intra.

## 7. Recomendação Final (3-way)
- [x] **LIMITES ATINGIDOS:** A esteira de aceleração Vulkan Compute para HEVC atingiu o teto arquitetural viável na AMD BC-250. A GPU opera com sucesso nas 40 CUs e a CPU mantém a estabilidade das árvores NAL e CABAC sem perda de quadros.

## 8. Recomendação de PR
- [x] **PR #1 (ME):** Aberto e ativo em https://github.com/simpmix/bc250-encoding-decoding-fix/pull/60.
- [x] **PR #2 (Pipeline Completo):** Aberto e ativo em https://github.com/simpmix/bc250-encoding-decoding-fix/pull/61 com o commit de correção `1819bfc` integrado.
