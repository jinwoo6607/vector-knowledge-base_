# vector-knowledge-base_

-- 1. 테이블 생성 
CREATE TABLE IF NOT EXISTS healthcare_knowledge (
  id BIGSERIAL PRIMARY KEY,           -- 고유 번호
  category TEXT NOT NULL,             -- 대분류 (diet, maintenance, fitness, spine, joint, alcohol 등)
  subcategory TEXT,                   -- 소분류 (emergency, exercise, lifestyle 등)
  content TEXT NOT NULL,              -- 실제 가이드라인 내용
  tags TEXT[],                        -- 검색용 태그 (배열 형태)
  created_at TIMESTAMPTZ DEFAULT NOW() -- 등록 시간
);

---(수정가능)
-- 사용자의 인바디 수치를 저장하는 테이블입니다.
CREATE TABLE IF NOT EXISTS inbody_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id), -- 로그인한 사용자의 ID와 연결
  
  -- OCR로 추출할 핵심 수치들
  weight FLOAT,                  -- 체중
  skeletal_muscle_mass FLOAT,    -- 골격근량
  body_fat_percentage FLOAT,     -- 체지방률
  bmi FLOAT,                     -- BMI
  
  image_url TEXT,                -- 나중에 이미지를 다시 볼 수 있게 스토리지 주소 저장
  created_at TIMESTAMPTZ DEFAULT NOW() -- 기록 날짜
);
---- 인바디 로그 테이블을 완전히 삭제합니다.(ocr안할시수정가능부분부터삭제필요)
-----DROP TABLE IF EXISTS inbody_logs;

-- 2. 데이터 적재 (요청하신 순서대로 정렬됨)

-- [1] 다이어트(Diet) 가이드라인
INSERT INTO healthcare_knowledge (category, subcategory, content, tags)
VALUES 
('diet', 'calorie_logic', '하루 권장 칼로리에서 500kcal를 감량하는 것이 주당 0.5kg 감량에 가장 안전하며, 건강 유지를 위해 최소 1,200kcal 이상 섭취를 유지해야 함', ARRAY['calorie', 'weight_loss', 'safety']),
('diet', 'macronutrients', '비만 단계에서는 탄수화물 30%, 단백질 40%, 지방 30%의 저탄수화물 고단백 식단이 효율적이며, 근육 성장을 위해서는 단백질 비중을 높임', ARRAY['macro', 'protein', 'carbs']),
('diet', 'meal_timing', '채소 -> 단백질 -> 탄수화물 순서로 먹는 거꾸로 식사법은 혈당 스파이크를 방지하고 인슐린 분비를 조절해 체지방 축적을 막음', ARRAY['blood_sugar', 'insulin', 'habit']),
('diet', 'hydration', '식사 30분 전 물 섭취는 포만감을 유도해 과식을 방지하며, 하루 2L 내외의 수분 섭취는 신진대사를 촉진함', ARRAY['water', 'metabolism', 'hydration']),
('diet', 'warning', '원푸드 다이어트나 극단적인 단식은 근손실과 요요 현상을 유발하므로 금지하며, 영양 균형이 잡힌 규칙적인 식사를 권장함', ARRAY['warning', 'yoyo', 'safety']);

-- [2] 건강 유지(Maintenance) 가이드라인
INSERT INTO healthcare_knowledge (category, subcategory, content, tags)
VALUES 
('maintenance', 'energy_balance', '현재 체중 유지를 위해 에너지 섭취량과 소모량이 평형을 이루는 TDEE(일일 총 에너지 소비량) 수준의 식단을 권장함', ARRAY['homeostasis', 'balance', 'weight_control']),
('maintenance', 'exercise_consistency', '체력 유지를 위해 중강도 유산소 운동 주 150분 또는 고강도 운동 주 75분을 권장하며, 근손실 방지를 위해 주 2회 전신 근력 운동 병행', ARRAY['consistency', 'health_habit', 'cardio']),
('maintenance', 'balanced_nutrition', '탄수화물 50%, 단백질 20~30%, 지방 20~30%의 균형 잡힌 비율을 유지하며, 가공식품보다는 자연식 위주의 식습관을 지속할 것', ARRAY['balanced_diet', 'nutrition', 'standard']),
('maintenance', 'monitoring_habit', '주 1회 체중 측정 및 월 1회 인바디 측정을 통해 신체 변화를 모니터링하고, 급격한 변화(±3kg) 발생 시 식단과 운동 강도를 즉시 재조정함', ARRAY['monitoring', 'inbody', 'self_care']),
('maintenance', 'lifestyle_recovery', '하루 7시간 이상의 질 좋은 수면과 충분한 수분 섭취, 스트레스 관리를 통해 신체 회복력을 유지하는 것이 장기적인 건강 유지의 핵심임', ARRAY['recovery', 'sleep', 'stress_management']);

-- [3] 근육 키우기(Muscle Gain/Fitness) 가이드라인
INSERT INTO healthcare_knowledge (category, subcategory, content, tags)
VALUES 
('fitness', 'hypertrophy_principle', '근육 성장을 위해 주 3~5회 웨이트 트레이닝을 실시하며, 매주 중량이나 횟수를 조금씩 늘리는 점진적 과부하 원칙을 반드시 적용해야 함', ARRAY['muscle_gain', 'overload', 'strength']),
('fitness', 'hypertrophy_rep_range', '근비대를 위해 8~12회 반복 가능한 중량으로 세트당 1~2분 휴식을 권장하며, 마지막 횟수에서 근육의 한계에 도달하는 강도가 가장 효율적임', ARRAY['reps', 'hypertrophy', 'size']),
('diet', 'protein_standard', '근육 합성을 위해 체중 1kg당 1.6g~2.0g의 단백질 섭취를 권장하며, 운동 직후 30분~2시간 이내에 양질의 단백질과 적절한 탄수화물을 섭취할 것', ARRAY['protein', 'diet', 'bulking']),
('fitness', 'muscle_recovery', '근육은 운동 중이 아니라 휴식 중에 성장하므로, 동일 부위 운동 후 48시간의 회복 시간을 두고 하루 7~8시간의 충분한 수면을 취해야 함', ARRAY['recovery', 'sleep', 'growth']),
('fitness', 'compound_movements', '스쿼트, 데드리프트, 벤치프레스 등 여러 관절을 사용하는 다관절 복합 운동 위주로 루틴을 구성하면 전신 근육량 증대와 호르몬 분비에 유리함', ARRAY['big3', 'compound', 'efficiency']);

-- [4] 척추(Spine) 일반 가이드라인
INSERT INTO healthcare_knowledge (category, subcategory, content, tags)
VALUES 
('spine', 'emergency', '대소변 장애, 마비 증상, 심한 야간 통증(VAS 7점 이상) 발생 시 즉시 병원 방문 권고', ARRAY['red_flag', 'emergency', 'safety']),
('spine', 'cervical', 'C자 커브 유지, 모니터 눈높이 조절, 맥켄지 신전 운동 수시 실시', ARRAY['neck', 'turtle_neck', 'posture']),
('spine', 'lumbar', '요추 전만(C자 곡선) 유지, 물건 들 때 무릎 굽히기, 윗몸일으키기 등 허리 굽히는 동작 금지', ARRAY['back', 'lumbar', 'disc']),
('spine', 'exercise', '걷기, 수영(자유형/배영), 플랭크와 같은 정적 코어 안정화 운동 권장', ARRAY['exercise', 'rehab', 'walking']),
('spine', 'habit', '의자 앉을 때 허리 쿠션 사용, 50분마다 스트레칭, 옆으로 잘 때 무릎 사이 베개 사용', ARRAY['lifestyle', 'habit', 'sitting']);

-- [5] 디스크(Disc) 정밀 가이드라인
INSERT INTO healthcare_knowledge (category, subcategory, content, tags)
VALUES 
('spine', 'disc_emergency', '마미증후군(대소변 장애), 발가락/발목 마비(근력 저하), 극심한 안정 시 통증 발생 시 즉시 응급실/전문의 방문 권고', ARRAY['emergency', 'red_flag', 'cauda_equina']),
('spine', 'disc_forbidden', '윗몸일으키기, 허리 숙여 발 끝 닿기, 무거운 데드리프트 등 허리를 앞으로 강하게 굽히는 동작 절대 금지', ARRAY['forbidden', 'prohibited', 'safety']),
('spine', 'disc_exercise', '맥켄지 신전 운동(허리 펴기), 평지 걷기, 수영(자유형/배영), 데드버그 및 버드독 코어 안정화 운동 권장', ARRAY['exercise', 'rehab', 'walking']),
('spine', 'disc_lifestyle', '양반다리 금지, 장시간 좌식 생활 피하기(50분마다 휴식), 허리 쿠션 사용 및 무릎 굽혀 물건 들기 습관화', ARRAY['habit', 'lifestyle', 'sitting']),
('spine', 'disc_acute', '통증이 심한 급성기에는 운동보다 요추 전만(C자 곡선)을 유지한 채 충분한 침상 휴식이 우선임', ARRAY['acute', 'rest', 'recovery']);

-- [6] 관절(Joint) 가이드라인
INSERT INTO healthcare_knowledge (category, subcategory, content, tags)
VALUES 
('joint', 'emergency', '관절 잠김(Locking), 부종 및 열감, 관절 불안정증, VAS 6점 이상의 심한 통증 시 즉시 전문의 진료 권고', ARRAY['emergency', 'safety', 'pain']),
('joint', 'knee', '쪼그려 앉기, 계단 내려가기, 줄넘기 금지. 평지 걷기, 수영, 실내 자전거 권장 및 대퇴사두근 강화 필수', ARRAY['knee', 'walking', 'quads']),
('joint', 'shoulder', '팔을 뒤로 과하게 꺾거나 무거운 짐 들기 금지. 가동 범위 내 스트레칭 및 회전근개 강화 운동 실시', ARRAY['shoulder', 'stretching', 'rotator_cuff']),
('joint', 'ankle', '울퉁불퉁한 길 달리기, 높은 굽 신발 피하기. 한 발로 서기 균형 잡기 및 발가락으로 수건 당기기 운동 권장', ARRAY['ankle', 'balance', 'stability']),
('joint', 'exercise', '저충격(Low-impact) 운동 선택, 통증 없는 가동 범위(ROM) 내 실시, 주변 근육 강화를 통한 하중 분산', ARRAY['exercise', 'safety', 'low_impact']);

-- [7] 음주(Alcohol) 가이드라인
INSERT INTO healthcare_knowledge (category, subcategory, content, tags)
VALUES 
('alcohol', 'standard', '남성 하루 2잔 이하, 여성 하루 1잔 이하 권장. 남성 5잔/여성 3잔 이상은 과음이며, 주 2회 이상 과음 시 고위험군으로 분류', ARRAY['risk_level', 'standard', 'safety']),
('alcohol', 'prohibited', '고혈압약, 당뇨약, 소염진통제 복용 시 간 손상 및 혈당 쇼크 위험. 급성 염증이나 척추 통증 시 알코올은 증상을 악화시키므로 금주 필수', ARRAY['medicine', 'emergency', 'prohibited']),
('alcohol', 'bmi_interaction', '알코올은 1g당 7kcal의 고열량이며 체지방 연소를 방해함. 특히 내장지방(복부 비만)의 주원인이므로 다이어트 시 최우선 절제 대상', ARRAY['bmi', 'obesity', 'diet']),
('alcohol', 'spine_interaction', '알코올은 디스크의 수분을 빼앗아 탄력을 줄이고 신경 주변의 부종(부기)을 악화시켜 척추 통증을 심화시킴', ARRAY['spine', 'inflammation', 'disc']),
('alcohol', 'tips', '음주 시 술 한 잔당 물 두 잔 마시기, 빈속에 마시지 않기, 단백질과 채소 위주의 안주를 먼저 섭취하여 알코올 흡수 늦추기', ARRAY['tips', 'habit', 'lifestyle']);

-- [8] 알러지 및 운동 유발 반응 정밀 가이드라인 (최종 통합 수정본)

INSERT INTO healthcare_knowledge (category, subcategory, content, tags)
VALUES 
('allergy', 'fdeia_detail', '음식 의존성 운동 유발성 아나필락시스(FDEIA) 예방: 밀가루, 메밀, 땅콩, 갑각류 등 본인 확인된 유발 식품 섭취 후 최소 4시간 이내에는 중강도 이상의 유산소 및 근력 운동 금지. 초기 증상(두드러기, 가려움) 발생 시 즉시 중단', ARRAY['FDEIA', 'safety', 'timing']),
('allergy', 'outdoor_pollen_peak', '꽃가루/황사 시즌 야외 운동 제한: 오전 6시~10시는 꽃가루 농도가 가장 높으므로 해당 시간대 실외 러닝 금지. 습도가 높거나 비 온 직후는 상대적으로 안전하나, 대기질 지수(AQI) 100 이상 시 무조건 실내 운동(홈트레이닝)으로 전환 권장', ARRAY['pollen', 'outdoor_limit', 'timing']),
('allergy', 'contact_material_pro', '접촉성 알러지 장비 관리: 라텍스 알러지 시 고무 저항 밴드 대신 TPE 소재 밴드 또는 덤벨/머신 활용. 니켈 알러지 의심 사용자는 도금되지 않은 우레탄 덤벨이나 가죽 장갑을 착용하여 장비 접촉 차단 유도', ARRAY['latex_free', 'nickel_allergy', 'equipment']),
('allergy', 'eib_warmup', '운동 유발성 기관지 수축(EIB) 관리: 찬 공기 노출 시 기도 수축 위험이 크므로 실외 운동 시 마스크 착용 필수. 본 운동 전 10~15분간 저강도 인터벌 웜업을 실시하여 기도의 ''내성 기간(Refractory Period)''을 확보하는 루틴 구성', ARRAY['asthma', 'EIB', 'warm_up']),
('allergy', 'medication_impact', '항히스타민제 복용 시 신체 반응 조절: 1세대 약물 복용 시 중추신경 억제로 인한 반사 신경 저하 및 입마름 발생. 수분 섭취량을 평소보다 20% 늘리고, 고중량 스쿼트/데드리프트 등 낙상 위험이 있는 종목은 50% 이하 저강도로 조정', ARRAY['medication', 'dehydration', 'intensity']);



--Supabase pgvector에 임베딩하여 적재 with python
-- healthcare_knowledge 테이블에 'embedding'이라는 이름의 숫자 보관함을 만듦
--CREATE EXTENSION IF NOT EXISTS vector;
--ALTER TABLE healthcare_knowledge ADD COLUMN IF NOT EXISTS embedding vector(1536);


--임베딩
ALTER TABLE healthcare_knowledge DROP COLUMN IF EXISTS embedding;
--(Gemini) 전용 768차원
ALTER TABLE healthcare_knowledge ADD COLUMN embedding vector(768);

-- 제미나이 768차원 규격에 맞추어 생성.
-- [추가] 검색 함수 생성
CREATE OR REPLACE FUNCTION match_healthcare_knowledge (
  query_embedding vector(768),
  match_threshold float,
  match_count int
)
RETURNS TABLE (
  id bigint,
  category text,
  subcategory text,
  content text,
  similarity float
)
LANGUAGE plpgsql
AS $$
BEGIN
  RETURN QUERY
  SELECT
    hk.id,
    hk.category,
    hk.subcategory,
    hk.content,
    1 - (hk.embedding <=> query_embedding) AS similarity
  FROM healthcare_knowledge hk
  WHERE 1 - (hk.embedding <=> query_embedding) > match_threshold
  ORDER BY similarity DESC
  LIMIT match_count;
END;
$$;
[basicguidline.sql](https://github.com/user-attachments/files/26674551/basicguidline.sql)
