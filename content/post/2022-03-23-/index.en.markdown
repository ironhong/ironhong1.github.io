---
title: Quasi-experimental methods- Propensity Score Matching
author: 'Iron Hong'
date: '2022-03-23'
slug: ''
categories: []
tags: [PSM, Stata, R]
subtitle: ''
summary: ''
authors: [Iron Hong]
lastmod: '2022-03-23T17:33:12+09:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---




# Introduction

일반적으로 임상의학에서는 주로 진정한 실험설계(true experimental design)를 통해서 프로그램(ex. covid-19 백신)의 효과에 인과적 추론(causal inference)을 합니다. 특히 무작위 통제 실험(RCT: Randomized Controlled Trial)이라는 기법을 활용해서 실험집단(treatment group)과 통제집단(control group)을 동질하게 구성하는 것인데, 무작위 배정(randomization) 조건을 만족해야 실험집단에 실시한 프로그램이 두 집단의 결과차이를 발생시키는 진정한 원인이라고 할 수 있습니다(See Angrist and Pischke, 2008)).

그러나 무작위 배정의 실험연구를 통해 데이터를 수집하는 것은 현실적으로 금전적이나 시간적 측면에서 보면 결코 쉬운 일은 아닙니다. 특히 사회과학에서는 윤리문제 이외에도 정치, 사회, 경제적 환경 등의 여러 가지 제약 때문에 무작위 배정이 만족하지 못하는 실험연구나 관찰(observation)자료를 이용하는 연구들로 많이 이뤄지고 있습니다. 문제는 이러한 연구들은 제3의 요인, "교란요인"(confounder)에 의해서 프로그램 뿐만 이니라 결과변수(혹은 종속변수)와도 연관되어 있어 프로그램과 결과변수 간의 인과관계(causality)를 왜곡시킬 수 있습니다.    

예를 들면, 어떤 연구자는 음주와 폐암의 관계를 분석하기 위해 건강한 통제집단(일반인)과 실험집단(폐암환자)을 구성했습니다. 각 대상자들에게 과거 음주 경험을 조사해서 음주가 폐암에 유의미한 영향을 미치는 것을 발견했는데, 이는 과연 정확한 인과관계라고 할 수 있을까요? 정답은 아닐 수 있습니다. 옆에 <Fig. 1>을 살펴본 바와 같이, 폐암을 유발하는 또 하나의 중요한 요인, 흡연의 영향을 충분히 고려하지 않았기 때문입니다. 다시 말해, 흡연이 폐암뿐만 아니라 음주에도 영향을 미칠 수 있기 때문에 연구대상의 선정과정에서 이른바 "선택 편의"(selection bias)가 발생해서 부적절한 연구결과를 도출할 수 있습니다. 

이러한 경우, 사회과학에서는 준실험 설계(quasi experimental design) 방법으로 주로 매칭(matching) 등을 활용해서 교란요인에 의한 추정 편의 문제를 '해결'^[여기서 완전한 해결보다는 일정 수준의 해소 또는 완화함을 뜻합니다.]합니다. 매칭은 여러 개의 핵심 공변량(key covariates)들을 매칭변수(matching variables)로 설정해서 실험집단과 통제집단을 최대한 동질적으로 구성하도록 연구대상들을 서로 매칭하는 것을 의미합니다. 매칭의 핵심 내용은 전체 분석대상에서 실험집단의 특성과 가장 유사한 특성을 가진 통제집단의 대상자를 찾아내서 매칭하는 과정입니다. 여기서 가장 중요한 점은 바로 어떻게 또는 어떤 방식으로 실험집단과 비슷한 통제집단을 찾아내는 것입니다. 현재 매칭 방법에 대한 다양한 연구가 활발하게 진행되고 있는 가운데, 대표적인 매칭 방법은 성향점수 매칭(Propensity Score Matching), 층화 매칭(Stratification Matching), 엔트로피 균형 매칭(Entropy Balance Matching) 등이 있습니다^[이 글에서는 성향점수 매칭 기법을 중심으로 논의하고자 합니다.]. 

# Propensity Score Matching
성향점수 매칭(PSM)은 Harvard 교수 Rubin(1974, 1978)과 Rosennbaum and Rubin(1983)이 제시한 이후, 사회과학에서 다양한 주제에서 프로그램의 효과 추정에 사용되고 있습니다. PSM의 핵심은 각 연구대상(개인, 도시, 국가 등)이 실험집단에 속하느냐의 멤버십(membership)을 결정하는 처치 방정식(treatment equation)을 설정한 다음에, 매칭변수를 통해 실험집단에 속할 확률을 예측함으로써 실험집단에 속한 각 연구대상에 매칭되는 반사실적(counterfactual) 집단을 만들게 됩니다.
성향점수는 옆에 수식과 같이 처리 여부(혹은 참여 여부)를 결정하는$X_{i}$ 매칭변수가 주어졌을 때 실험집단에 속할 조건부 확률(conditional probability)로 정의됩니다. 

Rosennbaum and Rubin(1983)은 매칭변수인 공변량에 기반해서 단위들을 일치시키는 결과와 성향점수에 기반해서  일치시키는 결과에 동일한 효과를 갖고 있음을 밝혔습니다. 특히 성향점수는 공변령의 대부분의 정보를 담고 있어 공변량  전체 집합(모든 변수 투입) 대신 단일 변수(성향점수)를 기준으로 매칭시키기 때문에 차원 축소 측면에서도 장점이 있는 것으로 알려져 있습니다. 

이러한 성향점수에 기반해서 프로그램의 효과 추정을 위해서는 1) 조건부 독립성 가정(CIA: Conditional Independence Assumption), 2) 공통지지영역 가정(CSA: Common Support Assumption)을 충족해야 합니다.
