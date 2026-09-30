# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 14:15–17:00

**Domain:** OrbitTech Store Customer Support

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 14:15–14:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (14:30–14:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric            | Acceptable Low Score Scenario                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Critical Low Score Scenario                                                                                                                                                                                                                                                                                                                                                                                                                 | Action Required                                                                                                                                                                                                                                                                                                                                                                                           |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Faithfulness      | Score thấp có thể tạm chấp nhận khi câu trả lời có một số diễn giải, tổng hợp hoặc suy luận đơn giản không xuất hiện nguyên văn trong context nhưng vẫn được hỗ trợ hợp lý bởi thông tin đã retrieve. Ví dụ, hệ thống tổng hợp nhiều đoạn tài liệu thành một kết luận chung hoặc diễn đạt lại nội dung bằng cách khác. Trong các ứng dụng có mức rủi ro thấp như chatbot FAQ, hỗ trợ học tập hoặc gợi ý nội dung, một mức giảm nhẹ về Faithfulness có thể được chấp nhận nếu thông tin chính vẫn đúng và không gây hiểu nhầm nghiêm trọng. | Score thấp trở nên critical khi mô hình tạo ra thông tin không có trong context, đưa ra số liệu, sự kiện hoặc kết luận sai, hoặc mâu thuẫn trực tiếp với tài liệu được cung cấp. Đây là dấu hiệu hallucination. Đặc biệt trong các lĩnh vực như y tế, tài chính, pháp lý hoặc hệ thống hỗ trợ ra quyết định, Faithfulness thấp có thể dẫn đến hậu quả nghiêm trọng vì người dùng có thể tin vào thông tin không được nguồn tài liệu hỗ trợ. | Kiểm tra các câu trả lời có hallucination; cải thiện prompt để yêu cầu mô hình chỉ sử dụng thông tin trong context; yêu cầu trích dẫn hoặc chỉ ra nguồn cho các khẳng định; cải thiện chất lượng retrieval; loại bỏ context mâu thuẫn; kiểm tra lại chunking và reranking. Nếu Faithfulness dưới 0.6, cần ưu tiên điều tra ngay vì đây thường là một trong những metric quan trọng nhất của hệ thống RAG. |
| Answer Relevance  | Score thấp có thể chấp nhận khi câu trả lời vẫn giải quyết đúng câu hỏi nhưng có thêm một số thông tin phụ, giải thích dài hơn cần thiết hoặc cung cấp thêm bối cảnh cho người dùng. Ví dụ, người dùng hỏi định nghĩa một thuật ngữ nhưng hệ thống vừa giải thích định nghĩa vừa đưa thêm ví dụ. Trong trường hợp này, câu trả lời vẫn hữu ích mặc dù chưa hoàn toàn tập trung vào câu hỏi.                                                                                                                                                | Score thấp trở nên critical khi câu trả lời không giải quyết intent chính của người dùng, trả lời sang một chủ đề khác, hiểu sai câu hỏi hoặc chỉ đưa ra thông tin liên quan rất ít đến yêu cầu ban đầu. Ví dụ, người dùng hỏi cách xử lý một lỗi cụ thể nhưng hệ thống chỉ giải thích lý thuyết chung mà không đưa ra cách xử lý. Khi đó, người dùng không nhận được giá trị thực tế từ hệ thống.                                          | Phân tích các query có Answer Relevance thấp để kiểm tra xem hệ thống có hiểu sai intent hay không. Cải thiện prompt, query rewriting hoặc query classification; yêu cầu mô hình trả lời trực tiếp câu hỏi trước rồi mới bổ sung giải thích; loại bỏ các phần lan man; cải thiện retrieval để context phù hợp hơn với câu hỏi.                                                                            |
| Context Recall    | Score thấp có thể tạm chấp nhận khi retriever không lấy được toàn bộ các tài liệu liên quan nhưng những chunk được retrieve vẫn chứa đủ thông tin để mô hình tạo ra câu trả lời chính xác và đầy đủ. Ví dụ, có 5 đoạn tài liệu liên quan nhưng chỉ retrieve được 3 đoạn, tuy nhiên 3 đoạn này đã chứa toàn bộ thông tin cần thiết để trả lời câu hỏi.                                                                                                                                                                                      | Score thấp trở nên critical khi retriever bỏ sót các tài liệu hoặc chunk chứa thông tin quan trọng, khiến mô hình không có đủ dữ liệu để trả lời đúng. Điều này có thể dẫn đến câu trả lời thiếu ý, sai hoặc buộc mô hình phải suy đoán. Đặc biệt với câu hỏi nhiều phần hoặc câu hỏi cần thông tin nằm ở nhiều tài liệu khác nhau, Context Recall thấp có thể ảnh hưởng trực tiếp đến chất lượng cuối cùng của câu trả lời.                | Kiểm tra lại retrieval pipeline; tăng hoặc điều chỉnh top-k; cải thiện embedding model; sử dụng hybrid search kết hợp semantic search và keyword search; áp dụng query expansion hoặc query rewriting; tối ưu kích thước chunk và chunk overlap; kiểm tra metadata filtering. Nếu cần, sử dụng multi-query retrieval để tăng khả năng tìm thấy các tài liệu liên quan.                                    |
| Context Precision | Score thấp có thể chấp nhận khi retriever lấy thêm một số chunk không liên quan nhưng các chunk quan trọng vẫn xuất hiện ở vị trí cao và mô hình vẫn có khả năng sử dụng đúng thông tin cần thiết. Ví dụ, hệ thống retrieve 5 chunk, trong đó 3 chunk rất liên quan và 2 chunk hơi dư thừa. Trường hợp này chủ yếu làm tăng chi phí token nhưng chưa gây ảnh hưởng lớn đến chất lượng câu trả lời.                                                                                                                                         | Score thấp trở nên critical khi phần lớn context được retrieve không liên quan đến câu hỏi, trong khi các tài liệu quan trọng bị đẩy xuống cuối hoặc không xuất hiện. Context nhiễu có thể khiến LLM chọn nhầm thông tin, hiểu sai câu hỏi hoặc tạo câu trả lời không chính xác. Ngoài ra, retrieval quá nhiều dữ liệu không liên quan còn làm tăng latency, chi phí token và có thể vượt quá context window của mô hình.                   | Phân tích thứ hạng các tài liệu được retrieve; giảm hoặc tối ưu giá trị top-k; sử dụng reranker để đưa tài liệu quan trọng lên đầu; áp dụng metadata filtering; cải thiện embedding; tối ưu chunking; loại bỏ các tài liệu trùng lặp hoặc không liên quan. Có thể áp dụng similarity threshold để loại những chunk có độ liên quan quá thấp.                                                              |
| Completeness      | Score thấp có thể chấp nhận khi câu trả lời chỉ thiếu một vài chi tiết phụ nhưng vẫn giải quyết được mục tiêu chính của câu hỏi. Ví dụ, người dùng hỏi ba ưu điểm chính của một kỹ thuật và câu trả lời trình bày đầy đủ các ưu điểm quan trọng nhưng thiếu một ví dụ minh họa. Nếu phần bị thiếu không ảnh hưởng đến khả năng hiểu hoặc sử dụng câu trả lời thì mức giảm nhẹ có thể được chấp nhận.                                                                                                                                       | Score thấp trở nên critical khi câu trả lời bỏ sót một hoặc nhiều phần quan trọng của câu hỏi. Điều này thường xảy ra với các câu hỏi nhiều yêu cầu, chẳng hạn người dùng yêu cầu vừa giải thích khái niệm, vừa đưa ví dụ, vừa so sánh ưu nhược điểm nhưng hệ thống chỉ trả lời một phần. Completeness thấp cũng có thể xuất phát từ việc retriever không lấy đủ context hoặc model không sử dụng toàn bộ thông tin đã được cung cấp.       | Phân tích xem thông tin bị thiếu nằm ở retrieval hay generation. Nếu context đã đầy đủ nhưng answer vẫn thiếu, cần cải thiện prompt để yêu cầu mô hình kiểm tra và trả lời từng phần của câu hỏi. Nếu context bị thiếu thì cần cải thiện Context Recall. Có thể tách câu hỏi nhiều ý thành các sub-question, retrieve riêng từng phần rồi tổng hợp câu trả lời cuối cùng.                                 |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Chọn một tập câu hỏi và, với mỗi câu hỏi, chuẩn bị hai câu trả lời A và B đã
> được human đánh giá trước. Giữ nguyên model judge, prompt, rubric, temperature
> và nội dung câu trả lời; chỉ thay đổi thứ tự trình bày:
>
> - **Condition 1:** đưa cho judge theo thứ tự `[A, B]` và yêu cầu chọn câu trả
>   lời tốt hơn hoặc chấm điểm từng câu.
> - **Condition 2:** đưa đúng cặp đó theo thứ tự `[B, A]`.
>
> Chạy trên nhiều cặp và đổi thứ tự ngẫu nhiên để tránh ảnh hưởng của từng câu
> hỏi riêng lẻ. Sau đó so sánh tỷ lệ vị trí thứ nhất được chọn và tỷ lệ judge
> đảo lựa chọn khi hoán đổi thứ tự. Nếu cùng một answer được ưu tiên trong cả
> hai conditions thì kết quả ổn định; nếu vị trí thứ nhất được chọn nhiều hơn
> đáng kể (ví dụ kiểm định ghép cặp cho thấy khác biệt có ý nghĩa), dù nội dung
> không đổi, đó là bằng chứng của position bias. Human labels đóng vai trò mốc
> để tránh nhầm position bias với việc A thực sự tốt hơn B.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> Rubric phải tách **độ đúng, độ đầy đủ, độ liên quan và tính súc tích** thành
> các tiêu chí độc lập, mô tả rõ bằng các mức điểm có thể quan sát được. Rubric
> cần nói rõ rằng độ dài tự nó không được cộng điểm: câu trả lời ngắn nhưng bao
> phủ đủ các ý bắt buộc phải nhận điểm tương đương câu dài; thông tin lặp lại,
> lan man hoặc không giúp giải quyết câu hỏi phải bị trừ ở tiêu chí relevance/
> conciseness. Có thể cung cấp checklist các ý bắt buộc và ví dụ neo điểm
> (anchor) gồm một đáp án ngắn-đủ và một đáp án dài nhưng nhiều nội dung thừa.
> Khi chấm tổng, không dùng số từ làm tín hiệu chất lượng và nên giới hạn trọng
> số của style để nội dung chính xác, có bằng chứng vẫn quyết định kết quả.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> Vì điểm của LLM judge không mặc nhiên tương ứng với tiêu chuẩn chất lượng mà
> con người mong muốn. Judge có thể hiểu rubric khác người chấm, chấm quá dễ
> hoặc quá nghiêm và chịu position, verbosity hay self-preference bias. So sánh
> điểm judge với một tập human labels đáng tin cậy giúp đo agreement, phát hiện
> loại case bất đồng có hệ thống, điều chỉnh prompt/rubric/threshold và biết khi
> nào phải chuyển sang human review. Việc hiệu chuẩn cũng làm cho quality gate
> trong CI/CD có ý nghĩa: ngưỡng 0.8 chỉ nên dùng để ra quyết định sau khi đã
> chứng minh rằng nó tương ứng với mức chất lượng được con người chấp nhận.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric           | Threshold | Lý do |
| ---------------- | --------: | ----- |
| Faithfulness     |       0.80 | Đây là tiêu chí an toàn quan trọng nhất của RAG; dưới ngưỡng có nguy cơ answer chứa claim không được context hỗ trợ. |
| Answer Relevance |       0.70 | Cho phép một ít thông tin bổ sung nhưng vẫn yêu cầu answer giải quyết đúng intent chính của khách hàng. |
| Completeness     |       0.70 | Cho phép thiếu chi tiết phụ, nhưng dưới mức này có nguy cơ bỏ sót bước hoặc điều kiện quan trọng. |

> Deployment bị block nếu **bất kỳ điểm trung bình nào trên regression set**
> thấp hơn ngưỡng tương ứng. Ngoài ra, một critical test case có Faithfulness
> dưới 0.60 cũng block dù điểm trung bình vẫn đạt, vì average có thể che khuất
> hallucination nghiêm trọng. Các ngưỡng phải được hiệu chuẩn bằng baseline và
> human labels, rồi phiên bản mới còn phải được kiểm tra để không regression so
> với phiên bản đang chạy.

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> - **Offline evaluation:** dùng trước khi merge/deploy và trong regression
>   test trên golden dataset cố định. Nó phù hợp để so sánh model, prompt,
>   retriever hoặc cấu hình một cách nhanh, lặp lại được và không gây rủi ro cho
>   người dùng thật. Hạn chế là dataset có thể không phản ánh hết traffic thực.
> - **Online evaluation:** dùng sau khi đã qua offline gate, triển khai bằng
>   canary/A-B test và giám sát trên traffic thật. Theo dõi feedback, task
>   success, escalation rate, latency, cost và các sự cố an toàn để phát hiện
>   distribution shift hay vấn đề chỉ xuất hiện trong thực tế. Cần guardrail,
>   rollout nhỏ và rollback tự động khi metric xấu đi.
> - **Human review:** dùng để tạo và kiểm tra golden labels, calibrate LLM judge,
>   phân xử các case mà metric/judges bất đồng, và đánh giá câu hỏi mơ hồ hoặc
>   có rủi ro cao (bảo mật, thanh toán, ngoại lệ chính sách). Human review cũng
>   cần cho mẫu ngẫu nhiên định kỳ của production nhằm phát hiện lỗi mà metric
>   tự động bỏ sót; không cần áp dụng cho mọi request vì chậm và tốn chi phí.

---

## Part 2 — Core Coding (14:45–15:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (15:40–16:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục                      | Kết quả       |
| ----------------------------- | ------------- |
| Tổng số records               | \_\_\_\_ / 20 |
| Easy                          | \_\_\_\_ / 5  |
| Medium                        | \_\_\_\_ / 7  |
| Hard                          | \_\_\_\_ / 5  |
| Adversarial                   | \_\_\_\_ / 3  |
| Source documents được sử dụng | \_\_\_\_ / 10 |
| Validator status              | PASS / FAIL   |

**Ba case đại diện cho quyết định thiết kế**

| ID  | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
| --- | ---------- | ------------------ | ----------------------------------------------- |
|     |            |                    |                                                 |
|     |            |                    |                                                 |
|     |            |                    |                                                 |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> _Câu trả lời:_

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID  | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
| --- | ---------------- | ---------: | ------------: | -----------: | --------: | -----------: | ------: | ------- | ------------ |
| E01 |                  |            |               |              |           |              |         |         |              |
| E02 |                  |            |               |              |           |              |         |         |              |
| E03 |                  |            |               |              |           |              |         |         |              |
| E04 |                  |            |               |              |           |              |         |         |              |
| E05 |                  |            |               |              |           |              |         |         |              |
| M01 |                  |            |               |              |           |              |         |         |              |
| M02 |                  |            |               |              |           |              |         |         |              |
| M03 |                  |            |               |              |           |              |         |         |              |
| M04 |                  |            |               |              |           |              |         |         |              |
| M05 |                  |            |               |              |           |              |         |         |              |
| M06 |                  |            |               |              |           |              |         |         |              |
| M07 |                  |            |               |              |           |              |         |         |              |
| H01 |                  |            |               |              |           |              |         |         |              |
| H02 |                  |            |               |              |           |              |         |         |              |
| H03 |                  |            |               |              |           |              |         |         |              |
| H04 |                  |            |               |              |           |              |         |         |              |
| H05 |                  |            |               |              |           |              |         |         |              |
| A01 |                  |            |               |              |           |              |         |         |              |
| A02 |                  |            |               |              |           |              |         |         |              |
| A03 |                  |            |               |              |           |              |         |         |              |

**Aggregate Report**

- Overall pass rate: \_\_\_\_%
- Avg Context Recall: \_\_\_\_
- Avg Context Precision: \_\_\_\_
- Avg Faithfulness: \_\_\_\_
- Avg Relevance: \_\_\_\_
- Avg Completeness: \_\_\_\_
- Failure type distribution: \_\_\_\_

**Ba cases có Overall Score thấp nhất**

1. ID: \_**\_ | Score: \_\_** | Failure type: \_\_\_\_
2. ID: \_**\_ | Score: \_\_** | Failure type: \_\_\_\_
3. ID: \_**\_ | Score: \_\_** | Failure type: \_\_\_\_

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> _Câu trả lời:_

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho OrbitTech Customer Support. Mỗi mức phải
đủ cụ thể để hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [ ] Correctness
- [ ] Completeness
- [ ] Relevance
- [ ] Evidence/citation
- [ ] Actionability
- [ ] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: \***\*\_\_\*\***

| Score | Tiêu chí domain-specific | Ví dụ response |
| ----: | ------------------------ | -------------- |
|     5 |                          |                |
|     4 |                          |                |
|     3 |                          |                |
|     2 |                          |                |
|     1 |                          |                |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
| --------- | ----------------- | --------------------- |
|           |                   |                       |
|           |                   |                       |
|           |                   |                       |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> _Câu trả lời:_

### Exercise 3.4 — Framework Comparison (Bonus +5)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí                  | Framework 1: \_\_\_\_ | Framework 2: \_\_\_\_ |
| ------------------------- | --------------------- | --------------------- |
| Setup complexity          |                       |                       |
| Metrics available         |                       |                       |
| CI/CD integration         |                       |                       |
| Kết quả trên cùng dataset |                       |                       |
| Insight rút ra            |                       |                       |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> _Phân tích:_

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID      | Recall before | Recall after | Precision before | Precision after | Delta Precision |
| ------- | ------------: | -----------: | ---------------: | --------------: | --------------: |
|         |               |              |                  |                 |                 |
|         |               |              |                  |                 |                 |
|         |               |              |                  |                 |                 |
|         |               |              |                  |                 |                 |
|         |               |              |                  |                 |                 |
| **Avg** |               |              |                  |                 |                 |

**Tại sao Recall dự kiến không đổi?**

> _Câu trả lời:_

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> _Câu trả lời:_

---

## Part 4 — Reflection (16:35–16:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 16:50–17:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
