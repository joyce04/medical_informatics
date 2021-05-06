[012619]

## Analysis of Sepsis with qSOFA.

지난번 회의에서 Infection 시점을 처방시점 데이터로 대체하는 가능성에 대해서 이야기했었고, 해당 데이터에 대한 토론을 기반으로 Sepsis를 판단하는 다양한 기준들이 있는데 [martin, angus, sofa, qsofa 등] 이중 저는 민혁님이 알려주신 qSOFA 기준을 mimic 데이터에서 찾을 수 있는지 데이터적으로 탐색을 해보고자 합니다.

[먼저 qSOFA의 기준은 다음과 같습니다.](https://www.qsofa.org/#whatis)

1. Is the patient in the ICU?
2. Altered Mentation
3. Respiratory rate(breaths per minute)
4. Systolic blood pressure (mmHg)

동일 시점에서 위의 기준 값 전체를 고려하여 진단 판단이 되어야 합니다. 
**동일한 시점에 전체 데이터가 존재하는지는 조금 후에 고민하기로 하고 먼저 각 기준에 대한 데이터 혹은 상응하는 데이터를 찾아볼 예정입니다.**

1. 먼저, 민혁님이 추천해주신 기존 Sepsis 진단 방식들을 비교하는 논문의 SQL을 참고할 예정입니다.

   https://github.com/alistairewj/sepsis3-mimic/tree/master/query
   <br>

   a. 해당 논문은 처방레코드를 기반으로 Infection여부를 판단합니다. 

   - 항생제 View 구성. [https://github.com/alistairewj/sepsis3-mimic/blob/635200f26763dcf370bee8fbdc72aa477c035f10/query/tbls/abx-poe-list.sql]
     지난주에 제가 활용한 방법보다 더 상세하게 활용하고 있는데, 약물 타입과, route까지 고려해서 추출하고 있습니다.

   - 항생제 View와 icustays, microbiologyevents 테이블을 조인하여, 
     icustays intime(icu에 들어간 시간), icustays outime(icu에서 나온 시간)을 함께 고려하고,
     microbiology 관련 테스트나 sensitivity test 결과를 고려할 수 있도록 합니다.

     [https://github.com/alistairewj/sepsis3-mimic/blob/635200f26763dcf370bee8fbdc72aa477c035f10/query/tbls/abx-micro-prescription.sql]

     그리고 나서, infection 의심 환자군을 조성합니다. [https://github.com/alistairewj/sepsis3-mimic/blob/635200f26763dcf370bee8fbdc72aa477c035f10/query/tbls/suspicion-of-infection.sql]
     <br>

   b. chartevents 테이블을 활용해 Vital 관련 정보를 추출합니다.

   - chartevents와 infection 의심 환자군 View를 통해 [HeartRate, SysBP, DiasBP, MeanBP, RespRate, TempF, TempC, SpO2, Glucose] 를 위한 View를 생성합니다. [https://github.com/alistairewj/sepsis3-mimic/blob/635200f26763dcf370bee8fbdc72aa477c035f10/query/tbls/vitals-infect-time.sql]
     <br>

   c. Mentation 관련 부분인것 같은데, icustays테이블을 활용해서, [GCSVerbal, GCSMotor, GCSEyes, Verbal Response, Motor Response, Eye Opening] 정보를 추출한 View를 생성합니다. [https://github.com/alistairewj/sepsis3-mimic/blob/635200f26763dcf370bee8fbdc72aa477c035f10/query/tbls/gcs-infect-time.sql]


2. 위의 쿼리를 참고해서, 추가적으로 데이터 분석을 진행했습니다.

   a. 앞선 논문에서 활용한 항생제 Route와 지난 중에 정의한 항생제 목록으로(infection에 의한 항생제 목록을 추린 내역이라 이 방식이 조금 더 환자군을 명확히 잡아 줄 것이라고 생각했습니다.) 만약 angus_sepsis에서 infection=1이었다면 다음과 같은 방식으로 환자군을 찾을 수 있을 것이라고 생각됩니다.

   ```sql
   SELECT count(*)
   FROM (
          SELECT
            prescriptions.subject_id,
            prescriptions.hadm_id,
            drug,
            drug_name_generic,
            route,
            min(startdate)
          FROM prescriptions, (SELECT *
                               FROM angus_sepsis
                               WHERE infection = 1) AS angus_sep
          WHERE prescriptions.hadm_id = angus_sep.hadm_id
                AND drug_type IN ('MAIN', 'ADDITIVE')
                AND drug ~*
                    '(Amikacin|Amoxicillin|Ampicillin|Azithromycin|Aztreonam|Vancomycin
                    |Cefaclor|Cefadroxil|Cephalexin|Cefamandole|Cefepime|Cefixime|Cefot
                    axime|Cefoxitin|Cefpodoxime|Cefprozil|Ceftazidime|Ceftriaxone|Cefur
                    oxime|Chloramphenicol|Ciprofloxacin|Clarithromycin|Clindamycin|Coli
                    stin|Dapsone|Daptomycin|Dicloxacillin|Doripenem|Doxycycline|Ertapen
                    em|Erythromycin|Ethambutol|Flucloxacillin|Fosfomycin|Gatifloxacin|G
                    eldanamycin|Gentamicin|Imipenem|Isoniazid|Kanamycin|Levofloxacin|Li
                    nezolid|Meropenem|Methicillin|Metronidazole|Minocycline|Moxifloxaci
                    n|Trovafloxacin|Nafcillin|Nalidixic
                    acid|Tobramycin|Netilmicin|Nitrofurantoin|Norfloxacin|Ofloxacin|Par
                    omomycin|Penicillin|Piperacillin|Polymyxin|Pyrazinamide|Quinupristi
                    n|Rifabutin|Rifampicin|Rifampin|Spectinomycin|Streptomycin|Sulfadia
                    zine|Sulfamethoxazole|Telithromycin|Tetracycline|Ticarcillin|Tigecy
                    cline|Trimethoprim)'
                -- we exclude routes via the eye, ears, or topically
                AND route NOT IN ('OU', 'OS', 'OD', 'AU', 'AS', 'AD', 'TP')
                AND route NOT ILIKE '%ear%'
                AND route NOT ILIKE '%eye%'
                -- we exclude certain types of antibiotics: topical creams, gels, desens, etc
                AND drug NOT ILIKE '%cream%'
                AND drug NOT ILIKE '%desensitization%'
                AND drug NOT ILIKE '%ophth oint%'
                AND drug NOT ILIKE '%gel%') AS infection_drugs;
   ```

   처방 시작 시간이 가장 빠른 시간를 찾기 위해 입원 아이디를 기준으로 그룹화한다면 **19236** 레코드가 추출됩니다.

   ```sql
   SELECT count(*)
   FROM (
          SELECT
            prescriptions.hadm_id,
            min(startdate)
          FROM prescriptions, (SELECT *
                               FROM angus_sepsis
                               WHERE infection = 1) AS angus_sep
          WHERE prescriptions.hadm_id = angus_sep.hadm_id
                AND drug_type IN ('MAIN', 'ADDITIVE')
                AND drug ~*
                    '(Amikacin|Amoxicillin|Ampicillin|Azithromycin......  너무 길어서...)'
                -- we exclude routes via the eye, ears, or topically
                AND route NOT IN ('OU', 'OS', 'OD', 'AU', 'AS', 'AD', 'TP')
                AND route NOT ILIKE '%ear%'
                AND route NOT ILIKE '%eye%'
                -- we exclude certain types of antibiotics: topical creams, gels, desens, etc
                AND drug NOT ILIKE '%cream%'
                AND drug NOT ILIKE '%desensitization%'
                AND drug NOT ILIKE '%ophth oint%'
                AND drug NOT ILIKE '%gel%'
          GROUP BY prescriptions.hadm_id) AS infection_drugs;
   ```



   **하지만 기존에는 angus를 기준으로 환자군을 구성한 내역이었어서, 해당 뷰를 활용하지 않고 infection을 위해 처방된 약물 레코드를 정의하려고 해보았습니다.**

   ```sql
   WITH prescrip AS (
       SELECT *
       FROM prescriptions
       WHERE drug_type IN ('MAIN', 'ADDITIVE')
             AND drug ~*
                 '(Amikacin|Amoxicillin|Ampicillin|Azithromycin......  너무 길어서...)'
             -- we exclude routes via the eye, ears, or topically
             AND route NOT IN ('OU', 'OS', 'OD', 'AU', 'AS', 'AD', 'TP')
             AND route !~* '(ear|eye)'
             -- we exclude certain types of antibiotics: topical creams, gels, desens, etc
             AND drug !~* '(cream|desensitization|ophth oint|gel)'
   )
   SELECT
     prescrip.hadm_id,
     prescrip.drug,
     prescrip.drug_type,
     prescrip.drug_name_generic,
     prescrip.route,
     prescrip.startdate
   FROM
     (SELECT
        hadm_id,
        min(startdate) AS startdate
      FROM prescrip
      GROUP BY hadm_id) AS p1
     LEFT JOIN prescrip ON p1.hadm_id = prescrip.hadm_id
                           AND p1.startdate = prescrip.startdate;
   ```

   위의 논문에서는 처방뿐만 아니라 microbiologyevents테이블을 활용해서 Positive Culture 여부를 확인합니다. (Positive Culture가 결국 혈액을 활용해서 infection 여부를 판단하는 것이라는 정도로만 이해했습니다.)[https://www.healthline.com/health/blood-culture]

   ```sql
   SELECT
     hadm_id,
     chartdate,
     charttime,
     spec_type_desc,
     max(CASE WHEN org_name IS NOT NULL AND org_name != ''
       THEN 1
         ELSE 0 END) AS PositiveCulture
   FROM microbiologyevents
   WHERE hadm_id = 191488
   GROUP BY hadm_id, chartdate, charttime, spec_type_desc;
   ```

   예시로 다음과 같은 입원아이디 191488의 환자를 보시면 Positiveculture 여부를 확인할 수 있습니다.

   | hadm_id | chartdate       | charttime        | spec_type_desc | positiveculture |
   | ------- | --------------- | ---------------- | -------------- | --------------- |
   | 191488  | 11/09/2184 0:00 | 11/09/2184 15:25 | BLOOD CULTURE  | 0               |
   | 191488  | 11/09/2184 0:00 | 11/09/2184 15:29 | BLOOD CULTURE  | 0               |
   | 191488  | 11/09/2184 0:00 | 11/09/2184 15:22 | SPUTUM         | 0               |

   참고한 논문에서는 처방시간 72시간, 24시간 앞을 확인하는데요, 이유를 파악하지 못했습니다. 만약 Positive culture가 발견되었고 처방 시간보다 빠르다면 해당 시간을 활용하면 될 것 같은데요...



Reference : 

- https://github.com/alistairewj/sepsis3-mimic