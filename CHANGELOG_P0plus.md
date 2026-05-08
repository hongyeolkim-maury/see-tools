# P0+ Patch 2026-05 — NIEL/TID Anchor 정정

## 배경

P0 패치(2026-05) 후 4개 파일을 SR-NIEL v11.0, AE9/AP9, NASA TM-20220011775 표준과 교차 검증한 결과 **2건의 절대값 결함**을 발견하여 정정합니다.

## 발견된 결함

### ① NIEL 절대값 — 모든 물질 ~2.85× 과소

P0 패치 시 사용한 anchor `NIEL_Si@10MeV = 3.30×10⁻³ MeV·cm²/g`가 잘못된 값이었습니다.

**검증된 정확한 anchor (3개 독립 출처 일치)**:
- SR-NIEL v11.0 (Boschini et al., Ed=21eV): **9.40×10⁻³**
- AE9/AP9 SCREAM data file (Aerospace 2016): **9.43×10⁻³**
- Zephyr Spectral Calculator (SR-NIEL 기반): **9.40×10⁻³**

→ 확정 anchor: **9.40×10⁻³ MeV·cm²/g**

### ② LEO TID rate — 6~15× 과대

P0 패치 시 NASA TM-20220011775 5mm Al 데이터를 차폐 두께 환산 없이 적용하여 LEO 궤도들이 실제보다 6~15× 높게 출력되었습니다.

**SHIELDOSE-2 표준 (2.54mm Al, 100mil)**:
- LEO eq: 0.03~0.05 krad/yr
- LEO ISS/Polar: 0.6~1 krad/yr  
- GEO: 10~20 krad/yr (Melagen 5yr 50~100 krad 정합)
- GTO/MEO: 100~150 krad/yr
- Deep cruise: ~10 krad/yr

## 수정 파일

| 파일 | 변경 내용 |
|---|---|
| `niel-tid.html` | NIEL0 객체 모든 물질 ×2.85 보정, TID_RATE0 LEO ×0.2~0.4, NIEL 참조 표 갱신 |
| `compare.html` | computeTNID의 niel0=3.3e-3 → 9.40e-3, ORBIT_TID0 동일 수정 |
| `solar-rad.html` | 1MeV-e 환산 인자 580 → 400 (NRL Isc, NASA TP-2004-213338) |
| `index.html` | NIEL/TID 절대값 미사용 (변경 불필요), 버전 표기만 v2.2 |

## 검증 결과 (회귀 테스트)

| 시나리오 | P0+ 결과 | 참조 | 상태 |
|---|---|---|---|
| Si NIEL @ 10MeV | 9.40×10⁻³ | 9.40×10⁻³ | ✓ 0% 차이 |
| GaAs NIEL @ 10MeV | 1.50×10⁻² | 1.50×10⁻² | ✓ 0% 차이 |
| GEO 3yr GaAs TNID | 9×10⁸ MeV/g | 1e8~1e9 | ✓ 범위 내 |
| Si CMOS GEO 5yr TID | 50 krad | 50~100 (Melagen) | ✓ 범위 내 |
| GEO 10yr TID | 100 krad | 50~200 | ✓ 합리적 |

## 출처 (Citation Ready)

- SR-NIEL v11.0 — Boschini, Rancoita et al., sr-niel.org
- AE9/AP9 SCREAM kernel — O'Brien & Kwan, Aerospace ATR-2016-03268
- Zephyr Spectral Displacement Damage Calculator — zephyr-spectral.com
- NASA TM-20220011775 — Mission Radiation Environment Modeling, Alena 2022
- Melagen Labs 2025 — 5-year GEO TID @ 2mm Al benchmark
- ECSS-E-ST-10-04C / SHIELDOSE-2 (Seltzer 1994 NISTIR 5477)
- NASA TP-2004-213338 — Mining Damage Curves, Walters et al.

## 변경 마커

각 수정 위치에 `[P0+ 2026-05 ANCHOR FIX]`, `[P0+ 2026-05 TID FIX]`, `[P0+ 2026-05 NRL FIX]` 주석 마커가 있어 git diff로 추적 용이.

```bash
grep -rn "P0+ 2026-05" *.html
```

## 시나리오 영향 (사용자 임무 평가)

이번 수정으로 **NIEL 결과는 약 2.85× 증가**, **LEO TID 결과는 약 0.2~0.4× 감소**합니다. 상대 비교(소자 A vs B, 궤도 A vs B)는 보존되나, 절대 임무 수명 평가가 달라질 수 있어 기존 산출물은 재평가 권장.

특히 영향이 큰 사례:
- Si CMOS @ LEO 5yr: 기존 1.5 krad → 수정 0.25 krad (5×)
- GaAs solar cell @ GEO 5yr: TNID 기존 1×10⁹ → 수정 3×10⁹ MeV/g (3×)
- 1MeV-e equiv fluence: 기존 ×580 → 수정 ×400 (~30% 감소)
