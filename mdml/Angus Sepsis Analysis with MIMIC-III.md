Angus Sepsis Analysis with MIMIC-III

SQL Reference : https://github.com/MIT-LCP/mimic-code/blob/master/concepts/sepsis/angus.sql

| angus_sepsis |           |                 |                   |           |        |
| ------------ | --------- | --------------- | ----------------- | --------- | ------ |
| total        | infection | explicit_sepsis | organ_dysfunction | mech_vent | angus  |
| 58,976       | 21,948    | 4,085           | 20,564            | 14,545    | 15,254 |

위의 내용은 신영님이 공유해주신 내용과 동일합니다. 공유해주신 쿼리가 환자군을 워낙 잘 정리하고 있어서, 해당 쿼리를 그대로 활용해서 Materialized View를 생성했습니다.

1. 전체 레코드 (58,976) 중, subject_id(환자 아이디)를 기준으로 distinct값을 확인하면 46,520 record가 추출 됩니다. 여러번 입원한 환자들이 존재합니다.

   ```sql
   SELECT count(DISTINCT(subject_id)) from angus_sepsis;
   ```


2. 일단 간단하게 (hadm_id)입원 아이디를 기준으로 Exploratory하게 정리를 했습니다.
   데이터에 대한 이해도가 낮아서 이것저것 많이 시도를 한것이라, 불필요하다고 생각되는 부분은 뛰어넘어가시는게 좋을 것 같습니다.

   1. angus = 1의 경우 (15,254)

      - 최우선 diagnoses_icd : 한 입원 아이디마다 여러 진단에 대해 seq_num평가가 되어있는데, 최대 많은 seq_num은 39까지도 존재합니다. 하지만 진단이 등록(내려진) 시점에 대한 데이터가 없기 때문에 순서를 확인하는 것은 어렵다고 생각했습니다. 
        **여기서 고민이 많이 되기 시작한 부분이 Sankey chart를 시간에 따른 흐름으로 활용하는 방안을 생각하고 있었는데, 진단의 시간 순서를 정확히 알지 못한다면, 해당 차트로 표현하는 것 자체가 정확한 데이터 인지...**
        **저는 의료 전문 지식이 없다보니 진단명을 기준으로 더 심화되거나 그렇지 않은 경우를 판단하는 것이 가능해서 순서를 파악하는게 가능한지 모르겠습니다...**

        ```sql
        SELECT hadm_id, max(seq_num) as max_seq
        FROM diagnoses_icd
        WHERE hadm_id IN (SELECT hadm_id
                          FROM angus_sepsis
                          WHERE angus = 1)
        GROUP BY hadm_id
        ORDER BY max_seq DESC;
        ```

      -  일단 진단 중에 Infection으로 평가 할 수 있는 진단을 세부 진단들로 나눌 수 있지 않을까라고 생각했습니다.

        ```sql
        SELECT
          hadm_id,
          seq_num,
          short_title,
          icd_diagnoses.icd9_code
        FROM (
               SELECT
                 hadm_id,
                 seq_num,
                 icd9_code
               FROM diagnoses_icd
               WHERE hadm_id IN (SELECT hadm_id
                                 FROM angus_sepsis
                                 WHERE angus = 1 AND infection = 1)) AS diagnoses,
          (SELECT
             icd9_code,
             short_title
           FROM d_icd_diagnoses
           WHERE substring(icd9_code, 1, 3) IN ('001', '002', '003', '004', '005', '008', '009', '010', '011', '012', '013', '014', '015', '016', '017', '018', '020', '021', '022', '023', '024', '025', '026', '027', '030', '031', '032', '033', '034', '035', '036', '037', '038', '039', '040', '041', '090', '091', '092', '093', '094', '095', '096', '097', '098', '100', '101', '102', '103', '104', '110', '111', '112', '114', '115', '116', '117', '118', '320', '322', '324', '325', '420', '421', '451', '461','462', '463', '464', '465', '481', '482', '485', '486', '494', '510', '513', '540', '541', '542', '566', '567', '590', '597', '601', '614', '615', '616', '681', '682', '683', '686', '730')
                 OR SUBSTRING(icd9_code, 1, 4) IN ('5695', '5720', '5721', '5750', '5990', '7110', '7907', '9966', '9985', '9993')
                 OR SUBSTRING(icd9_code, 1, 5) IN ('49121', '56201', '56203', '56211', '56213', '56983')
          ) AS icd_diagnoses
        WHERE diagnoses.icd9_code = icd_diagnoses.icd9_code;
        ```

        위의 데이터를 icd9_code 기준 그룹으로 묶고 환자수가 높은 순위로 정렬하면 다음과 같습니다. (최대 랭킹 10까지만 표기했습니다.)

      - |      | icd9  | short_title              | hadm_count |
        | ---- | ----- | ------------------------ | ---------- |
        | 1    | 5990  | Urin tract infection NOS | 4513       |
        | 2    | 486   | Pneumonia, organism NOS  | 3477       |
        | 3    | 389   | Septicemia NOS           | 3289       |
        | 4    | 845   | Int inf clstrdium dfcile | 1112       |
        | 5    | 7907  | Bacteremia               | 1019       |
        | 6    | 49121 | Obs chr bronc w(ac) exac | 768        |
        | 7    | 99859 | Other postop infection   | 747        |
        | 8    | 48241 | Meth sus pneum d/t Staph | 692        |
        | 9    | 99662 | React-oth vasc dev/graft | 662        |
        | 10   | 6826  | Cellulitis of leg        | 491        |

      <br>

      - Organ dysfunction 도 동일하게 추출해보았습니다.

        |      | icd9  | short_title              | hadm_count |
        | ---- | ----- | ------------------------ | ---------- |
        | 1    | 5849  | Acute kidney failure NOS | 5906       |
        | 2    | 99592 | Severe sepsis            | 3651       |
        | 3    | 78552 | Septic shock             | 2586       |
        | 4    | 2875  | Thrombocytopenia NOS     | 1814       |
        | 5    | 5845  | Ac kidny fail, tubr necr | 1770       |
        | 6    | 4589  | Hypotension NOS          | 1044       |
        | 7    | 2930  | Delirium d/t other cond  | 926        |
        | 8    | 45829 | Iatrogenc hypotnsion NEC | 801        |
        | 9    | 570   | Acute necrosis of liver  | 754        |
        | 10   | 2869  | Coagulat defect NEC/NOS  | 683        |

        <br>

3. 반대로 angus = 0 인 경우,

   - Infection

     환자수를 기준으로 정렬했을때, 위의 angus=1일 때와 큰 차이가 없습니다.

   - Organ dysfunction

     |      | icd9  | short_title              | hadm_count |
     | ---- | ----- | ------------------------ | ---------- |
     | 1    | 5849  | Acute kidney failure NOS | 3213       |
     | 2    | 45829 | Iatrogenc hypotnsion NEC | 1320       |
     | 3    | 2875  | Thrombocytopenia NOS     | 1251       |
     | 4    | 4589  | Hypotension NOS          | 1007       |
     | 5    | 78551 | Cardiogenic shock        | 538        |
     | 6    | 2930  | Delirium d/t other cond  | 519        |
     | 7    | 5845  | Ac kidny fail, tubr necr | 517        |
     | 8    | 2869  | Coagulat defect NEC/NOS  | 333        |
     | 9    | 570   | Acute necrosis of liver  | 313        |
     | 10   | 34830 | Encephalopathy NOS       | 219        |

4. 다른 테이블을 확인해보기 전에 투약 테이블(prescriptions) 을 먼저 확인해서 투약 정보다 진단을 연결 할 수 있는 방법이 있을까라는 고민이 되었습니다. (이것은 그저 저의 가설..일 뿐입니다.)

   ```sql
   SELECT
     hadm_id,
     startdate,
     enddate,
     drug_type,
     drug,
     ndc,
     route
   FROM prescriptions
   WHERE hadm_id IN (SELECT hadm_id
                     FROM angus_sepsis
                     WHERE angus = 1 AND infection = 1)
   ORDER BY hadm_id, startdate;
   ```

   입원 중 infection 진단을 받고, angus 확진인 경우를 대상으로 해당 쿼리 형식으로 투약 정보를 살펴보기 시작했는데, 약의 종류가 사실 워낙 다양해서, infection과 연관된 정보를 찾아낼 수 있을지 조금 더 살펴봐야 할 것 같습니다.

