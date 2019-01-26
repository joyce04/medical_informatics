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




Reference : 

- https://github.com/alistairewj/sepsis3-mimic