# 資料庫 （PostgreSQL)

- ##### ERD：

````mermaid
graph LR
Teacher[教授] -- 1-擁有-N --> Course;
Course -- 1-擁有-N --> Problem[題目];
Problem -- 1-擁有-N --> Copy[抄襲];
Problem -- 1-擁有-N --> Judge;
Problem -- 1-擁有-N --> Team[隊伍];
Course -- 1-擁有-N --> Feedback;
Course[課程] -- 1-擁有-N --> Student_Course[學生_課程];
Student_Course -- N-擁有-1 --> Student[學生];
Student -- 1-擁有-N --> Feedback[回饋];
Student -- 1-擁有-N --> Judge[批改];
Course -- 1-擁有-N --> Assistant_Course[助教_課程];
Assistant_Course -- N-擁有-1 --> Assistant[助教];
ProblemBank
````

+ Table：

1. **person**：使用者

   ```json
   id // pk, bigserial
   account // text(not null), 學號
   password // text(not null), 密碼(加密)
   member_id // fk, bigserial 學生id || 助教id || 老師id || 管理員id
   ```

2. **role**：角色

   ```json
   id // pk, bigserial
   name // text(not null), 角色英文名稱
   name_zh // text(not null), 角色中文名稱
   ```

3. **person_role**：使用者與角色對應表

   ```json
   person_id // pk, bigserial
   role_id // pk, bigserial
   ```

4. **student**：學生

   ```json
   id // pk, bigserial
   name // text(not null), 學生姓名
   student_class // text(not null), 年度+學生系級 e.g.106資管B
   ```

5. **assistant**：助教

   ```json
   id // pk, bigserial
   name // text(not null), 助教姓名
   ```

6. **teacher**：老師

   ```json
   id // pk, bigserial
   name // text(not null), 老師姓名
   ```

7. **admin**：管理員

   ```json
   id // pk, bigserial
   name // text(not null), 管理員姓名
   ```

8. **course**：課程

   ```json
   id // pk, bigserial
   teacher_id // fk, bigserial
   course_name // text(not null), 課程名稱
   semester // text(not null), 課程日期(格式:104上 || 104下)
   ```

9. **student_course**：學生與課程對應表

   ```json
   student_id // pk, bigserial
   course_id // pk, bigserial
   ```

10. **assistant_course**：助教與課程對應表

   ```json
   assistant_id // pk, bigserial
   course_id // pk, bigserial
   ```

11. **problem**

    ```json
    id // pk, bigserial
    course_id // fk(not null), bigserial
    name // text(not null), 題目名稱
    type // text(not null), 題目類型(作業 || 練習題 || 討論題)
    category // text(not null), 題目作答類型(輸入輸出 || 輸入寫檔 || 讀檔輸出 || 讀檔寫檔)
    language // text[](not null), 題目可作答的程式語言(Java, Python)
    tag // text[](not null), 題型分類(條件(condition),迴圈(loop))
    rate // double precision(not null), 題目平均難易度,預設為0
    description // text(not null), 題目描述
    input_desc // text(not null), 輸入描述
    output_desc // text(not null), 輸出描述
    test_cases: [{ // json, 測試範本
    	"inputSample": "", // not null, 輸入範本
      	"outputSample": "" // not null, 輸出範本
    }]
    deadline // date(not null), 截止日期, 格式為yyyy-MM-dd
    correct_num // integer(not null), 此題正確(滿分)人數, 預設為0
    incorrect_num // integer(not null), 此題錯誤(未滿分)人數, 預設為0
    correct_rate // double precision(not null), 正確率, 預設為0
    best_student_id // text(null), 最佳代碼的學生id, 預設為null // 注意學生資料刪除時，設為null
    pattern // text[](not null), 該題須出現的模板代碼, 預設為空
    ```

12. **copy**

    ```json
    id // pk, bigserial
    problem_id // fk, bigserial
    student_one_id // foreign key, 學生1 id // 注意學生資料刪除時，須同步刪除copy
    student_two_id // foreign key, 學生2 id // 注意學生資料刪除時，須同步刪除copy
    similarity // double precision(not null), 兩者代碼相似度 
    ```

13. **judge**

   ```json
   id // pk, bigserial
   problem_id // fk, bigserial
   student_id // fk, bigserial
   rate // double precision, 題目難易度, 預設為0
   historyCode: {[ // json, 學生該題的歷史題交代碼
       "handDate": "", // String, 提交日期(格式:yyyy-MM-dd)
       "code": "", // String, 學生的代碼
       "runTime": 0, // double, 該代碼運行時間, 預設為0
       "output": [], // String, 輸出內容
       "error": [], // String, 錯誤訊息
       "symbol": [], // String, judge結果
       "score": 0, // double , 程式分數
   ]}
   ```

14. **team**(暫定)

   ```json
   id // primary key, bigserial
   problem_id // foreign key, bigserial
   account // text, 批改者學生的學號 // 刪除學生資料時，須同步刪除
   corrected_account // text[], 被批改者學生的學號 // 刪除學生資料時，須同步刪除
   comment_result:[{
     account: '', // double, 批改的學生學號 // 刪除學生資料時，須同步刪除
     correctValue: { // json, 程式正確性
        score: // double
        comment: // string
     },
     readValue:  { // json,  程式可讀性
        score: // double
        comment: // string
     },
     skillValue:  { //json, 技巧運用
        score: // double
        comment: // string
     }, 
     completeValue:  { // json, 程式完整性
        score: // double
        comment: // string
     }, 
     wholeValue:  { // json 綜合評分
        score: // double
        comment: // string
     }, 
     comment: // string 
   }]
   ```

15. **feedback**：意見回饋

   ```json
   id // pk, bigserial
   courseId // fk, bigserial
   studentId // fk, bigserial
   date // data, 提送feedback當時日期(格式yyyy-MM-dd)
   content // text, feedback內容
   ```

16. **problem_bank**：題庫

    ```json
    id // pk, bigserial
    course_id // fk(not null), bigserial
    name // text(not null), 題目名稱
    type // text(not null), 題目類型(作業 || 練習題 || 討論題)
    category // text(not null), 題目作答類型(輸入輸出 || 輸入寫檔 || 讀檔輸出 || 讀檔寫檔)
    language // text[](not null), 題目可作答的程式語言(Java, Python)
    tag // text[](not null), 題型分類(條件(condition),迴圈(loop))
    rate // double precision(not null), 題目平均難易度,預設為0
    description // text(not null), 題目描述
    input_desc // text(not null), 輸入描述
    output_desc // text(not null), 輸出描述
    test_cases: [{ // json, 測試範本
    	"inputSample": "", // not null, 輸入範本
      	"outputSample": "" // not null, 輸出範本
    }]
    ```

17. **dashboard(待做)**

    ```json
    id // primary key, bigserial
    onlineNum // integer, (今日)在線人數, 預設為0
    activeNum // integer, (今日)活躍人數，當天頁面訪問次數超過5(看checkLogin呼叫幾次), 預設為0
    todayDoDum // integer, 今日做題次數（今日批改次數), 預設為0
    weekDoNum // integer, 本週做題次數（本週批改次數）, 預設為0
    onlineData: {[ // json 在線與活躍人數線圖(上限：6天), 預設為空陣列
    	"date": "", // String, 日期(格式:yy-mm-dd)
         "onlineNum": 0, // integer, 日期當天在線人數
         "activeNum": 0 // integer, 日期當天活躍人數
    ]},
    doPerDayData: [{ // json 每日做題次數(上限：6天), 預設為空陣列
            "date": "", // String, 日期(格式:yyyy-mm-dd)
            "doPerDayNum": 0 // integer, 當日做題次數, 預設為0 
    }],
    judgeLiveData: [{ // json, 即時批改實況(上限：10則), 預設為空陣列
            "studentId": int, // String, 學生Id 
            "problemId": int, // String, 題目Id
    }]
    ```

# Restful API

- URL = https://hostname:8081/api/

- Response Format

  result：回傳的資料

- Restful Status Code

  | Status Code | Description                                                  |
  | ----------- | ------------------------------------------------------------ |
  | 200         | 請求成功，並且有返回內容(GET)                                |
  | 201         | 請求成功，並且在伺服器上了新建或修改資源(POST/PUT/PATCH)     |
  | 204         | 請求成功，但是沒有需要返回內容(DELETE)                       |
  | 400         | 請求失敗，使用者發出的請求有錯誤，伺服器没有進行新建或修改資源的操作 |
  | 401         | 請求失敗，使用者需要登入才可執行該動作                       |
  | 403         | 請求失敗，使用者沒有權限可以執行該動作                       |
  | 404         | 請求失敗，資源、檔案不存在                                   |
  | 500         | 請求失敗，伺服器出現錯誤                                     |

1. 基礎api

   | API Method | API URL    | Desc | Req Params        | Resp Result                                                  |
   | ---------- | ---------- | ---- | ----------------- | ------------------------------------------------------------ |
   | POST       | URL/login  | 登入 | account, password | authority(判斷身份。student \|\| teacher \|\| assistant \|\| admin) |
   | POST       | URL/logout | 登出 |                   |                                                              |
   

   
2. 學生api（student）

   - [x] 更改個人密碼
   - [x] 取得所有課程
   - [x] 取得課程下的個人學生資料
   - [x] 取得課程下的學生歷史成績及題目資訊
   - [x] 取得課程下的學生題目資訊(可根據題目類型、作答於否取得)
   - [x] 更新課程下的學生對題目的難易度評分

   | API Method | API URL                              | Desc                           | Req Params                                                   | Resp Result                                                  |
   | ---------- | ------------------------------------ | ------------------------------ | ------------------------------------------------------------ | :----------------------------------------------------------- |
   | PUT        | URL/student/password                 | 更改密碼                       | oriPassowrd, newPassword                                     |                                                              |
   | GET        | URL/student/course                   | 學生全部課程                   |                                                              | [{ courseId, courseName, teacherName, semester }]            |
   | GET        | URL/student/course/profile           | 課程下的個人學生資料           | courseId                                                     | account, name, studentClass, undoNum, doneNum, bestCodeNum, correctNum, incorrectNum |
   | GET        | URL/student/course/problem/judgeInfo | 課程下的學生歷史成績及題目資訊 | courseId                                                     | [{problemId, problemName, type,  historyCode:[{handDate, code, runTime, output, symbol, score, errorMessage}], rate,  correctRate, isBestCode(Boolean), copys: [{account, similarity}]}] |
   | GET        | URL/student/course/problem           | 課程下的學生所有題目資料       | courseId, type:(作業 \|\| 練習題 \|\| 討論題 \|\| 全部), isJudge(boolean) | [{problemId, name, type, deadline, rate}]                    |
   | PUT        | URL/student/course/problem/rate      | 課程下的學生對題目的難易度評分 | problemId, rate                                              |                                                              |
   

   
3. 題目api（problem）

   - [x] 取得單一題目資訊
   - [x] 取得課程下的所有題目資訊
   - [x] 新增、編輯、刪除題目

   | API Method | API URL               | Desc                     | Req Params                                                   | Resp Result                                                  |
   | ---------- | --------------------- | ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | GET        | URL/problem           | 題目的資訊               | problemId                                                    | name, type, category, tag:[], rate, description, inputDesc, outputDesc, testCases: {inputSample, outputSample}, deadline, correctNum, incorrectNum, correctRate, pattern |
   | GET        | URL/problem/judgeInfo | 取得課程下的所有題目     | courseId                                                     | [{problemId, name, type, category, tag, status(判斷是否已過期。可作答、已關閉), undoStudentNum, doneStudentNum, rate, correctRate, deadline, bestStudentAccount, bestStudentName, pattern, bestRunTime}] |
   | GET        | URL/problem/score     | 取得題目下的所有學生成績 | problemId                                                    | [{studentAccount, studentName, studentClass, score, code}]   |
   | GET        | URL/problem/copy      | 取得題目下的抄襲         | problemId                                                    | [{studentOneAccount, studentOneName, studentTwoAccount, studentTwoName, similarity}] (如果沒有就回空List)}] |
   | POST       | URL/problem           | 新增題目                 | courseId, name, type, category, tag:[], description, inputDesc, ouputDesc, testCases:{inputSample, outputSample}, deadline,pattern | problemId                                                    |
   | PUT        | URL/problem           | 編輯題目                 | problemId, name, type, category, tag:[], rate, description, inputDesc, ouputDesc, testCases:{inputSample, outputSample}, deadline,  pattern |                                                              |
   | DELETE     | URL/problem           | 刪除題目                 | problemId                                                    |                                                              |

   

4. 批改api（judge）

   - [x] 批改代碼
   - [x] 取得被批改後的資訊
   - [x] 檢查此題目是否被批改過
   - [x] 判斷抄襲

   | API Method | API URL          | Desc                   | Req Params                                  | Resp Result                                                  |
   | ---------- | ---------------- | ---------------------- | ------------------------------------------- | ------------------------------------------------------------ |
   | POST       | URL/judge        | 批改代碼               | problemId, code, language(Java \|\| Python) | judgeId                                                      |
   | GET        | URL/judge        | 已被批改後的資訊       | judgeId                                     | handDate, score, runTime, code, symbol, output, errorInfo, best(Boolean) |
   | GET        | URL/judge/status | 學生在此題是否已被批改 | problemId                                   | judgeId(true 才有值 false: ""), judged(boolean)              |
   | POST       | URL/judge/copy   | 判斷抄襲               | problemId                                   |                                                              |

   

5. 教授Api(teacher)

   - [x] 建立、刪除課程
   - [x] 將學生加入、退出課程
   - [x] 將助教加入、退出課程
   - [x] 新增、編輯、刪除題目(參考Problem Api)
   - [x] 取得所有授課課程資訊(參考Course Api)
   - [x] 取得課程下的所有題目資訊(參考Course Api)
   - [x] 取得課程下的所有學生資訊(參考Course Api)
   - [x] 取得課程下的所有feedback(參考Feedback Api)
   - [ ] 建立、刪除隊伍(參考Team Api)

   | **API Method** | API URL                       | Desc                     | Req Params                                                  | Resp Result                                                  |
   | -------------- | ----------------------------- | ------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------ |
   | POST           | URL/teacher/course            | 建立課程                 | courseName, semester, studentClass, taList:[助教帳號]       |                                                              |
   | PUT            | URL/teacher/course            | 刪除課程                 | courseId                                                    |                                                              |
   | PUT            | URL/teacher/course/student    | 將學生加入課程           | courseId, action:( 0(加入) 1(退出)), accountList: [account] |                                                              |
   | PUT            | URL/teacher/course/assistant  | 將助教加入課程           | courseId,  action:( 0(加入) 1(退出), accountList:[account]  |                                                              |
   | GET            | URL/teacher/course            | 取得老師的所有課程       |                                                             | [{ courseId, courseName, teacherName, semester, class[], taList[] }] |
   | GET            | URL/teacher/studentClass      | 取得班級列表             |                                                             | [className]                                                  |
   | GET            | URL/teacher/unassignAssistant | 取得未被指派的助教名單   |                                                             | [{assistantId, assistantName}]                               |
   | GET            | URL/teacher/allStudentAccount | 取得課程下的所有學生學號 | courseId                                                    | [account]                                                    |
   | GET            | URL/teacher/allStudentProfile | 取得學號的學生資料       | [account]                                                   | [{account, name, studentClass}]                              |

   

6. 助教api (assistant)

   - [x] 將學生加入、退出課程
   - [x] 新增、編輯、刪除題目(參考Problem Api)
   - [x] 取得課程下的所有feedback (參考Feedback Api)
   - [ ] 建立、刪除隊伍(參考 Team Api)

   | API Method | API URL                         | Desc                     | Req Params                                                 | Resp Result                                                |
   | ---------- | ------------------------------- | ------------------------ | ---------------------------------------------------------- | ---------------------------------------------------------- |
   | PUT        | URL/assistant/course/student    | 將學生加入課程           | courseId,action:( 0(加入) 1(退出)), accountList: [account] |                                                            |
   | GET        | URL/assistant/course            | 取得助教的所有課程       |                                                            | [{ courseId, courseName, teacherName, semester, class[] }] |
   | GET        | URL/assistant/allStudentAccount | 取得課程下的所有學生學號 | courseId                                                   | [account]                                                  |
   | GET        | URL/assistant/allStudentProfile | 取得學號的學生資料       | [account]                                                  | [{account, name, studentClass}]                            |

7. 課程api(course)

   - [x] 取得所有課程資訊
   - [x] 取得課程裡的學生所有資訊

   | API Method | API URL                 | Desc                                                    | Req Params                               | Resp Result                          |
   | ---------- | ----------------------- | ------------------------------------------------------- | ---------------------------------------- | ------------------------------------ |
   | DELETE     | URL/course              | 刪除課程                                                | courseId                                 |                                      |
   | PUT        | URL/course              | 編輯課程                                                | courseId,   courseName, semester, taList |                                      |
   | GET        | URL/course/correctRank  | 取得正確解題的學生排行（前五名，有同值則同名，且值>0）  | courseId                                 | [{rank, account, name, correctNum}]  |
   | GET        | URL/course/bestCodeRank | 取得最佳解答的學生排行 （前五名，有同值則同名，且值>0） | courseId                                 | [{rank, account, name, bestCodeNum}] |
   | POST       | URL/course/feedback     | 新增回饋                                                | courseId, content                        |                                      |
   | GET        | URL/course/feedback     | 取得課程下的所有feedback                                | courseId                                 | [{account, name, date, content}]     |

8. 管理員api(admin)

  | API Method | API URL           | Desc           | Req Params                             | Resp Result |
  | ---------- | ----------------- | -------------- | -------------------------------------- | ----------- |
  | POST       | URL/admin/student | 加入學生至系統 | [{account,password,name,studentClass}] |             |

9. 題庫api

   | API Method | API URL                        | Desc                 | Req Params                                                   | Resp Result                                                  |
   | ---------- | ------------------------------ | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | POST       | URL/problemBank                | 在題庫中建立題目     | name, category, tag, description, inputDesc, outputDesc, testCases |                                                              |
   | PUT        | URL/problemBank                | 在題庫中編輯題目     | id, name, category, tag, description, inputDesc, outputDesc, testCases |                                                              |
   | GET        | URL/problemBank                | 在題庫中取得所有題目 |                                                              | [{id, name, category,  tag}]                                 |
   | GET        | URL/problemBank/:problemBankId | 取得題目詳細資訊     |                                                              | id, name, category, tag, description, inputDesc, outputDesc, testCases |
   | DELETE     | URL/problemBank                | 在題庫中刪除題目     | id                                                           |                                                              |

   

10. 隊伍api (team)(最後做)

   | API Method | API URL                        | Desc                           | Req Params                                                   | Resp Result                                                  |
   | ---------- | ------------------------------ | ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | POST       | URL/team                       | 建立討論題隊伍                 | problemId, pairs:[{correctAccount, correctedAccount}, ...]   |                                                              |
   | GET        | URL/team/correctStudents       | 取得此學生要批改的對象         | problemId                                                    | [{studentAccount, code}]                                     |
   | GET        | URL/team/correctStatus         | 取得此學生是否已經完成互評     | problemId                                                    | status(boolean)                                              |
   | GET        | URL/team/correctedStatus       | 取得此學生是否**被**完成互評   | problemId                                                    | status(boolean)                                              |
   | GET        | URL/team/correctInfo           | 取得此學生批改對方的資訊       | problemId                                                    | [{studentAccount, code, correctValue(程式正確性):{score, comment}, readValue(程式可讀性):{score, comment}, skillValue(技巧運用):{score, comment}, completeValue(程式完整性):{score, comment}, wholeValue(綜合評分):{score, comment}, comment(總評論))}] |
   | GET        | URL/team/correctedInfo         | 取得此學生批改對方的資訊       | problemId                                                    | [{correctValue(程式正確性), readValue(程式可讀性):{score, comment}, skillValue(技巧運用):{score, comment}, completeValue(程式完整性):{score, comment}, wholeValue(綜合評分):{score, comment}, comment(總評論)}] |
   | POST       | URL/team/commentResult         | 送出評分資訊                   | problemId, correctedList:[{correctedAccount(被批改的學號), correctValue:{score, comment}, readValue:{score, comment}, skillValue:{score, comment}, completeValue:{score, comment}, wholeValue:{score, comment}, comment(總評論)}] |                                                              |
   | GET        | URL/team/discussScore          | 取得互評成績                   | problemId                                                    | [{ account, name, studentClass, courseName, score(系統批改的成績), discussedScore(被批改的成績):[{studentAccount, correctValue:{score, comment}, readValue:{score, comment}, skillValue:{score, comment}, completeValue:{score, comment}, wholeValue:{score, comment}, comment(總評論)}] }] |
   | GET        | URL/team/teacher/correctInfo   | 老師取得此討論題學生的批改資訊 | problemId                                                    | [{studentAccount, code, isJudged(學生是否已送出), isTrJudged(老師是否已批改過),  correctValue:{score: "", comment: ""}, readValue:{score: "", comment: ""}, skillValue:{score: "", comment: ""}, completeValue:{score: "", comment: ""}, wholeValue:{score: "", comment: ""}, comment }] |
   | POST       | URL/team/teacher/commentResult | 老師批改此同學的討論題成績     | problemId, studentAccount, correctedList:[{correctedAccount(被批改的學號), correctValue:{score, comment}, readValue:{score, comment}, skillValue:{score, comment}, completeValue:{score, comment}, wholeValue:{score, comment}, comment(總評論)}] |                                                              |
