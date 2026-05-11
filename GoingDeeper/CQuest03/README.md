# AIFFEL Campus Online Code Peer Review Templete
- 코더 : 최병훈
- 리뷰어 : 정주열


# PRT(Peer Review Template)
- [x]  **1. 주어진 문제를 해결하는 완성된 코드가 제출되었나요?**
    - 문제에서 요구하는 최종 결과물이 첨부되었는지 확인
        - U-Net과 U-Net++ 성능을 비교하는 결과가 아래 이미지와 같이 잘 되어 있습니다.
    <img width="447" height="670" alt="image" src="https://github.com/user-attachments/assets/424606fd-a5fc-4be2-97f5-9f903262c7fd" />

- [x]  **2. 전체 코드에서 가장 핵심적이거나 가장 복잡하고 이해하기 어려운 부분에 작성된 
주석 또는 doc string을 보고 해당 코드가 잘 이해되었나요?**
    - 해당 코드 블럭을 왜 핵심적이라고 생각하는지 확인
    - 해당 코드 블럭에 doc string/annotation이 달려 있는지 확인
    - 해당 코드의 기능, 존재 이유, 작동 원리 등을 기술했는지 확인
    - 주석을 보고 코드 이해가 잘 되었는지 확인
        - 데이터 셋 만드는 부분에서 파라미터 설명과 각 함수의 기능을 주석으로 잘 설명되어 있어서 이해하기 쉬웠습니다.
    <img width="767" height="789" alt="image" src="https://github.com/user-attachments/assets/7daed176-dee4-470e-9e5e-314d9f2d0df8" />

- [x]  **3. 에러가 난 부분을 디버깅하여 문제를 해결한 기록을 남겼거나
새로운 시도 또는 추가 실험을 수행해봤나요?**
    - 문제 원인 및 해결 과정을 잘 기록하였는지 확인
    - 프로젝트 평가 기준에 더해 추가적으로 수행한 나만의 시도, 
    실험이 기록되어 있는지 확인
        - 아래 이미지와 같이 문제 원인을 추론하는 과정이 작성되어 있습니다. 또한 Deep supervision의 효과를 확인하기 위해서 비교 실험을 진행한 부분이 기록되어 있습니다.
    <img width="571" height="661" alt="image" src="https://github.com/user-attachments/assets/820af072-915e-4f97-855e-63c863a7acfd" />

- [x]  **4. 회고를 잘 작성했나요?**
    - 주어진 문제를 해결하는 완성된 코드 내지 프로젝트 결과물에 대해
    배운점과 아쉬운점, 느낀점 등이 기록되어 있는지 확인
    - 전체 코드 실행 플로우를 그래프로 그려서 이해를 돕고 있는지 확인
        - 최종 결론을 정리하면서 향후 계획을 수립하는 부분이 잘 작성되어 있습니다.
    <img width="583" height="473" alt="image" src="https://github.com/user-attachments/assets/bdaedccf-ff80-4cf3-8918-9c0006913feb" />

- [x]  **5. 코드가 간결하고 효율적인가요?**
    - 파이썬 스타일 가이드 (PEP8) 를 준수하였는지 확인
    - 코드 중복을 최소화하고 범용적으로 사용할 수 있도록 함수화/모듈화했는지 확인
        - 전체적으로 코드가 간결하고 효율적으로 작성되어 있습니다. 특히 아래 이미지에서 double_conv 기능이 UNet에서 많이 쓰이는데 함수화 해서 사용한 부분이 효율적이라 생각합니다.
    <img width="654" height="832" alt="image" src="https://github.com/user-attachments/assets/b6b98683-6994-435e-9469-ec78edfccf40" />


# 회고(참고 링크 및 코드 개선)
U-Net과 U-Net++를 비교한 내용이 깔끔하게 정리되어 있어서 읽기 수월했습니다. 또한 결과 부분을 따로 md파일로 나눠서 작성하신 점이 더 가독성이 좋았던 것 같습니다. 프로젝트 목표를 넘어서 Ablation study를 한 부분을 정말 인상깊게 봤습니다. 
