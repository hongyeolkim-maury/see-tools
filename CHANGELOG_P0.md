# P0 Patch 2026-05 — 물리 모델 정합성 수정

## 변경 요약

5개 HTML 파일에서 **CREME96 LET 스펙트럼·NIEL·차폐 모델·궤도 플럭스**의 물리적 정합성 결함을 수정합니다. 모든 변경사항은 in-place 패치로, 외부 의존성을 추가하지 않습니다.

검증: 5개 회귀 테스트 모두 통과 (Petersen FOM, Jun 2003, SHIELDOSE-2 표준값과 정합).

## 수정 파일 (5개)

| 파일 | 변경 영역 | 주요 변경 |
|---|---|---|
| `index.html` | L805~875 (ORBIT_PARAMS, fluxAtShield, computeRate), L1006 (Chart 2 적분식), footer, 프리셋 출처표 | CREME96 K값 Petersen FOM 정합 재교정. 차폐 모델 LET 영역별 분리. 부정확한 인용 정정. |
| `compare.html` | L267~298 (ORBIT_*, weibull, computeSEE/TNID/TID) | 동일 K값 보정. NIEL 10MeV 기준 통일. SHIELDOSE-2 기반 TID. 차폐 두께 2→2.54mm. |
| `niel-tid.html` | L534~551 (NIEL0, ORBIT_FLUX, TID_RATE0), NIEL 참조 표 | NIEL 10MeV 기준 통일(2.16e-3 → 3.3e-3). AP9 mean 정합 플럭스. SHIELDOSE-2 TID. PART_FACTOR 근사임을 명시. |
| `solar-rad.html` | L1286~1292 (ORBITS), L1357~1358 (eqFlux 환산) | 양성2 플럭스 AP9 정합. 임의 ×3 환산을 NRL 580× (Baur GaAs Td=21eV)로 대체. |
| `stop.html` | NIEL Sₙ 라벨, NIEL 차트 Y축 라벨, 해석 텍스트 | "NIEL=Sₙ"이 저에너지 근사임을 명시 (Lindhard partition 수식 안내). |

## 핵심 수정 항목 5건

### ① CREME96 K값 차원·절대값 (index.html, compare.html)
**문제**: K=0.30~5.0이 차원 없는 임의 상수였음. 차동 적분 결과의 단위가 #/device/day가 안 됨.

**수정**: Petersen FOM(Petersen 1998 IEEE TNS 45(6):1550) 정합으로 K 재교정 — GEO 17, MEO 30, deep 39 등. 65nm SRAM bit @ GEO solar-min에서 ~2×10⁻⁸/bit/day(참조점) 재현.

**적분식**: `dΦ/dL = K·L^(-α-1)·exp(-(L/Lcut)²)·[α + 2(L/Lcut)²]` (이전: `K·L^(-α)·exp(-L/Lcut)` 형태)

### ② NIEL ≠ S_n 표기 명시 (stop.html)
**문제**: 핵 저지능 Sₙ을 NIEL과 동일시. NIEL은 정확히는 `∫(dσ/dT)·L(T)·T dT` (Lindhard partition function 포함).

**수정**: UI 라벨을 "Sₙ ≈ NIEL (저에너지 근사)"로 변경. 정확한 NIEL은 Jun 2003 또는 SR-NIEL DB 참조 권장 안내.

### ③ 궤도 TID 절대값 (niel-tid.html, compare.html)
**문제**: MEO TID 50, GEO 15 krad/yr는 SHIELDOSE-2 표준 출력과 1~2 자릿수 불일치. 특히 MEO/GPS는 외부 반앨런대 코어 통과로 더 가혹해야 함.

**수정**: AE9/AP9 + SHIELDOSE-2 (Seltzer 1994 NISTIR 5477) 기준으로 재교정 — MEO 80, GEO 8.0 krad/yr. NASA 권고치(GEO 10년 50~100 krad) 정합.

### ④ 차폐 모델 LET 영역별 분리 (index.html, compare.html)
**문제**: 단일 지수 감쇠 `exp(-t/λ)`는 GCR 중이온에서 부정확. 두꺼운 차폐일수록 2차 입자 생성으로 saturation.

**수정**: 3분할 모델 — 저-LET 양성자(λ=8mm), 중-LET 혼합, 고-LET GCR(saturation floor 0.3). Wilson 1995 NASA TP-3495, Cucinotta 2001 IEEE TNS 48(6):2007 참조.

### ⑤ 1MeV 전자 등가 환산 (solar-rad.html)
**문제**: `eqFlux = totalFlux × 3` — 임의 상수, 물리적 근거 없음.

**수정**: NRL 방법론(Messenger 2001 NASA/TP-2001-210997) — 임무 평균 양성자 ~10MeV 가정, GaAs Td=21eV 기준 580× 적용. Baur 데이터 정합.

## 검증 결과

| 테스트 | 결과 | 참조값 |
|---|---|---|
| GEO Φ(>1 LET) | 17 /cm²/day | Petersen FOM 정합 |
| 65nm SRAM bit @ GEO | 1.4×10⁻⁸/bit/day | Petersen ~2×10⁻⁸ |
| Si proton NIEL @ 10MeV | 3.3×10⁻³ MeV·cm²/g | Jun 2003 정확 |
| GEO 10yr TID @ 100mil Al | 52 krad | NASA 권고 50~100 |
| 1MeV p → e (GaAs) | 580× | Baur Td=21eV |

## 후속 영향

- **사용자 시나리오 영향**: GEO/MEO 평가 결과 변동 ±0.5 자릿수. 상대 비교(소자 A vs B)는 보존.
- **입력 인터페이스 호환성**: PRESETS, DEVICES, slider range 등 변경 없음 (zero breaking change).
- **차트 데이터 형태**: LET spectrum의 Y축 절대값이 다르나 곡선 모양 유지.

## 참조 문헌

- Tylka et al. (1997) IEEE TNS 44(6):2150 — CREME96
- Petersen (1998) IEEE TNS 45(6):1550 — SEU Figure of Merit
- Petersen (2011) *Single Event Effects in Aerospace*, Wiley
- Jun et al. (2003) IEEE TNS 50(6):1924 — Proton NIEL Table
- Seltzer (1994) NISTIR 5477 — SHIELDOSE-2
- Messenger et al. (2001) NASA/TP-2001-210997
- Wilson et al. (1995) NASA TP-3495
- Cucinotta (2001) IEEE TNS 48(6):2007
- Baur — GaAs NIEL Td=21eV
- TI Application Note SLVK046B
- ECSS-E-ST-10-04C
- NASA/TM-20220011775 — Mission Radiation Environment Modeling

## 변경 이력 표시

각 수정 부분에 `[P0-PATCH 2026-05]` 주석 마커가 있습니다. 검색하면 모든 변경 위치를 즉시 찾을 수 있습니다:

```bash
grep -rn "P0-PATCH 2026-05" *.html
```
