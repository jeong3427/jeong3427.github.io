---
layout: post

title:  "json preset 내용과 문자열 비교"

categories : Python

tag : [python, json]

---




유저가 입력한 텍스트가, 기존에 설정해둔 텍스트와 같은 것인지 비교하는 팀장님 코드.

주요 내용만 기록.

```python
#콤마(,)가 분리자인 텍스트를 단어별로 끊는 함수
#괄호 안에서 ,가 쓰일 수도 있어서 ()에 대해 처리를 추가
def listify_prompts(prompts):
    prompt_list=[]
    seperator = False
    ready_to_pop = False
    stack=[]

    for c in prompts:
        if c=='(':
            stack.append(c)
        elif c==')':
            stack.append(c)
            ready_to_pop = True
        elif c==',':
            if len(stack)>0:
                if (stack[0]=='(' and ready_to_pop) or stack[0] != '(':
                    ready_to_pop=False
                    seperator=True
                    prompt_list.append("".join(stack).strip())
                    stack.clear()
        elif c== '' and seperator :
            seperator=False
        else:
            stack.append(c)

    if len(stack)>0:
        prompt_list.append("".join(stack).strip())

    return prompt_list


#te = "(abc, a, ded), ddd, eee"
#print(listify_prompts(te))
#['(abc a ded)', 'ddd', 'eee']


import json

preset_text = json.load(open('파일명.json', encoding='UTF8'))

preset = {}
input_text = {}

for i in preset_text:
    if i == 'variation':
        continue
    
    for first_category in preset_text[i]:
        cat1 = first_category['name_ko']
        
        for second_category in first_category['category']:
            cat2 = second_category['name_ko']
            
            for third_category in second_category['list']:
                cat3 = third_category['name_ko']
                
                preset[(cat1, cat2, cat3)] = [] 
                
                for input_text in listify_prompts(third_category['prompt']):
                    if input_text not in input_text:
                        input_text[input_text] = []                
                    input_text[input_text].append((cat1, cat2, cat3))
                    preset[(cat1, cat2, cat3)].append(input_text)



other_presets = listify_prompts("키워드1, 키워드2, 키워드3")
other_presets.extend(listify_prompts("키워드4, 키워드5, 키워드6"))
other_presets.extend(listify_prompts("키워드7, 키워드8, 키워드9"))

import pandas as pd

data_set = pd.read_csv('파일명')

results = []

for i, row in data_set.iterrows():
    result = {}
    # result['type'] = assign_result(row)
    
    user_prompts = listify_prompts(row['prompt'])
    user_neg_prompts = listify_prompts(row['negative_prompt'])

    # Prompt usage counting
    prompts_usage_count = {}
    prompts_usage_count_neg = {}
    user_added_prompts = []
    user_added_prompts_neg = []
    model_probs = {}
    model_neg_probs = {}
    preset_probs = {}

    if result['type'] == "a":
        for prompt in user_prompts:
            
            prompts_usage_count[prompt] = {'model':[], 'preset': [], 'face': False}
            for value in input_text:
                if value == prompt:
                    prompts_usage_count[prompt]['preset'].extend(input_text[value])
                    for i in input_text[value]:
                        if i not in preset_probs:
                            preset_probs[i] = 0
                        preset_probs[i] += 1                    
                    break

            for i in model_prompts:
                for value in model_prompts[i][0]:
                    if value == prompt:
                        prompts_usage_count[prompt]['model'].append(i)
                        if i not in model_probs:
                            model_probs[i] = 0
                        model_probs[i] += 1
                        break

            if prompt in other_presets:
                prompts_usage_count[prompt]['face'] = True
                continue

            if len(prompts_usage_count[prompt]['model']) == 0 and len(prompts_usage_count[prompt]['preset']) == 0:
                user_added_prompts.append(prompt)

        for prompt in user_neg_prompts:
            prompts_usage_count_neg[prompt] = {'model':[]}

            for i in model_prompts:
                for value in model_prompts[i][1]:
                    if value == prompt:
                        prompts_usage_count_neg[prompt]['model'].append(i)
                        if i not in model_neg_probs:
                            model_neg_probs[i] = 0
                        model_neg_probs[i] += 1
                        break
            if len(prompts_usage_count_neg[prompt]['model']) == 0:
                user_added_prompts_neg.append(prompt)



        #probs계산
        for i in preset_probs:
            preset_probs[i] = preset_probs[i] / len(preset[i])
            if preset_probs[i] > 1: 
                preset_probs[i] = 1

        for i in model_probs:
            if i in model_neg_probs:
                model_probs[i] = (model_probs[i] + model_neg_probs[i]) / (len(model_prompts[i][0]) + len(model_prompts[i][1]))
            else:
                model_probs[i] = model_probs[i] / len(model_prompts[i][0])
            if model_probs[i] > 1:
                model_probs[i] = 1
        for i in model_neg_probs:
            if i in model_probs:
                continue
            model_probs[i] = model_neg_probs[i] / len(model_prompts[i][1])
            if model_probs[i] > 1:
                model_probs[i] = 1

        preset_probs = dict(sorted(preset_probs.items(), key = lambda i: i[1], reverse=True))
        model_probs = dict(sorted(model_probs.items(), key = lambda i: i[1], reverse=True))

        preset_clicked = [ k[2] for k, v in preset_probs.items() if v == 1 ]
        preset_clicked = [ v if not v.startswith('AA ') else 'AA' for v in preset_clicked ]
        preset_clicked = ", ".join(list(dict.fromkeys(preset_clicked)))
        model_clicked = list(model_probs.keys())[0] if len(model_probs) > 0 else "-"

        if result['type'] == "b":
            for i in b_neg_prompt:
                if i in user_added_prompts_neg:
                    user_added_prompts_neg.remove(i)
        elif result['type'] == "c":
            for i in c_neg_prompt:
                if i in user_added_prompts_neg:
                    user_added_prompts_neg.remove(i)
        elif result['type'] == "d":
            for i in d_prompt:
                if i in user_added_prompts:
                    user_added_prompts.remove(i)

        result['preset'] = preset_clicked
        result['user_added_prompt'] = ", ".join(user_added_prompts)
        
    else:
        result['preset'] = ""
        result['user_added_prompt'] = ""
    # result['user_added_prompt_neg'] = ", ".join(user_added_prompts_neg)
    result['model'] = model_clicked
    result['w x h'] = str(row['result_width']) + " x " + str(row['result_height'])
    name = row['name'].split(" ") if type(row['name']) is str else "-"
    
    if len(name) > 1:
        team = str(" ".join(name[1:]))
        name = name[0]
    
    result['name'] = name
    result['team'] = "/".join(team) 
    result['job'] = team[3] if len(team) > 3 else "-"

    result['d'] = True if row['value.as'] == 'd' else False
    if result["job"] == "-":
        continue
    results.append(result)

```