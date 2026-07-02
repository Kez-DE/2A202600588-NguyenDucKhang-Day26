# Design — Hoàn thiện Lab Day 26 (MCP/A2A) đạt full điểm

**Ngày:** 2026-07-02
**Mục tiêu:** Hoàn thành toàn bộ 7 hạng mục bài tập trong `day26_mcp_a2a_lab.ipynb` để nộp bài đạt điểm tối đa.

## Hiện trạng

Code nền (4 agents, MCP server, governance, scripts) đã đầy đủ. Các bài tập chấm điểm chưa làm:

| Bài | Trạng thái | Loại |
|-----|-----------|------|
| 1.1 — Khám phá MCP server (3 câu hỏi) | Chưa trả lời | Lý thuyết |
| 1.2 — Tool `count_words` | Chưa làm | Code |
| 2.1 — Bảng A2A vs Sub-Agent Local | Chưa điền | Lý thuyết |
| 3.1 — `route_with_chain` | Chưa làm | Code |
| Capstone W1–W5 (ADK Web) | Placeholder, chưa chạy | Thực hành |
| 5.1 — Ma trận capability | Chưa viết | Lý thuyết |
| 5.2 — Mở rộng policy | 1/4 (synthesis_agent đã có sẵn) | Code |

Không có `.env` (chỉ `.env.example`) → không chạy được LLM flow trong môi trường này.

## Phạm vi & phân công

- **Claude:** toàn bộ code (1.2, 3.1, 5.2), toàn bộ lý thuyết (1.1, 2.1, 5.1), chạy các cell offline để notebook có output thật.
- **Sinh viên:** Phase 4 capstone — cần `GOOGLE_API_KEY`, chạy ADK Web, gõ 5 prompt, điền kết quả, chụp screenshot (W1, W2 tối thiểu).

## Thiết kế từng bài

### Bài 1.2 — `count_words`
- `mcp_server/research_tools_server.py`: thêm `Tool(name="count_words", inputSchema={text: string})` vào `list_tools()`; thêm nhánh `count_words` trong `call_tool()` trả `{"word_count": len(text.split())}`.
- `agents/orchestrator/agent.py`: thêm `count_words` vào `tool_filter`.
- `lab_utils/governance/policy.json`: khai báo tool `count_words` (guard từ chối tool không khai báo).
- Notebook: cell test gọi hàm trực tiếp, chạy có output.

### Bài 3.1 — `route_with_chain`
- `lab_utils/semantic_router.py`: method mới — thử `route()` chính; nếu điểm ≥ threshold trả về ngay; ngược lại duyệt `chain` theo thứ tự, trả agent đầu tiên có trong registry; hết chain trả phần tử cuối.
- Notebook: cell test với `chain=["search_agent", "database_agent", "orchestrator"]`, chạy có output.

### Bài 5.2 — Mở rộng governance
1. `synthesis_agent` trong `allowed_targets` — đã có sẵn, ghi chú xác nhận trong notebook.
2. `policy.json`: thêm `"blocked_keywords": ["password"]` cho `search_documents`; `guard.authorize_mcp_tool`: check keyword trong query → decision `deny`, ghi audit.
3. Chạy lại cell governance, xác nhận audit log có `deny` / `hitl_required`.
4. Test caller không hợp lệ: assert `authorize_mcp_connection("search_agent")` (hoặc caller lạ) trả deny — dạng cell notebook `assert`, không thêm framework test.

### Lý thuyết (1.1, 2.1, 5.1)
Viết trực tiếp vào cell markdown ngay dưới đề bài trong notebook, tiếng Việt, ngắn gọn đúng trọng tâm slide.

### Capstone (Phase 4 — sinh viên)
Checklist trong notebook đã có sẵn (cell 32–33). Bổ sung không cần thiết — chỉ cần chạy và điền. Screenshot W1 + W2 kèm khi nộp.

## Lộ trình

1. **Phase 1 — Code:** 1.2 → 3.1 → 5.2 (kèm verify offline từng bài).
2. **Phase 2 — Lý thuyết:** điền 1.1, 2.1, 5.1 vào notebook.
3. **Phase 3 — Verify & commit:** chạy các cell offline (router, governance, MCP direct) cho notebook có output; commit.
4. **Phase 4 — Capstone (sinh viên):** `.env` → `bash scripts/start_capstone.sh` → 5 prompt → điền `adk_web_results` → chạy cell → screenshot → commit nộp.

## Xử lý lỗi & kiểm thử

- Mọi thay đổi guard/policy phải giữ các cell governance hiện có chạy pass (không phá hành vi cũ).
- Test dạng `assert` trong notebook, không thêm pytest/framework — nhất quán với phong cách lab.
- Các phần cần API key được cô lập ở Phase 4; Phase 1–3 chạy hoàn toàn offline.

## Rủi ro

- W1–W5 phụ thuộc hành vi LLM và API key — có thể cần thử lại prompt vài lần (hướng dẫn đã có trong cell 32).
- Nếu guard có logic strict-tool-list, quên khai báo `count_words` trong policy sẽ khiến tool bị chặn — đã đưa vào thiết kế.
