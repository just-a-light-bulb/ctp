**API Request & Response**

คู่มือ Request & Response ของ API

Comic Trans Studio — Complete Endpoint Reference / เอกสารอ้างอิง API ฉบับสมบูรณ์

All endpoints return a standard envelope. / ทุก endpoint คืนค่าในรูปแบบมาตรฐาน:

// Standard response envelope — used by every endpoint

// รูปแบบ response มาตรฐาน — ใช้กับทุก endpoint

{

"ok": boolean, // true = success, false = error

"data": T | null, // payload on success / ข้อมูลเมื่อสำเร็จ

"error": string | null, // error code on failure / รหัสข้อผิดพลาด

"msg": string | null // human-readable message / ข้อความสำหรับแสดงผล

}

---

**Authentication / การยืนยันตัวตน:**

// Include the Kinde JWT in every protected API request:

// ส่ง Kinde JWT ใน Authorization header ทุกครั้งที่เรียก API ที่ต้องล็อกอิน:

Authorization: Bearer <kinde_access_token>

// The server hook in hooks.server.ts validates this token.

// หาก token ไม่ถูกต้องหรือหมดอายุ server จะตอบกลับด้วย HTTP 401.

---

▸ **Global Error Codes** *รหัสข้อผิดพลาดทั่วไป*

| **error code** | **HTTP** | **Message EN / ข้อความ TH** |
| --- | --- | --- |
| **UNAUTHORIZED** | **401** | Missing or invalid Authorization header.
*ไม่มีหรือ Authorization header ไม่ถูกต้อง* |
| **FORBIDDEN** | **403** | Authenticated but does not own this resource.
*ล็อกอินแล้วแต่ไม่มีสิทธิ์เข้าถึง resource นี้* |
| **NOT_FOUND** | **404** | The requested resource does not exist.
*ไม่พบ resource ที่ร้องขอ* |
| **VALIDATION_ERROR** | **422** | Request body failed validation. See msg for details.
*Request body ไม่ผ่านการตรวจสอบ ดู msg สำหรับรายละเอียด* |
| **RATE_LIMITED** | **429** | Too many requests. Retry after the Retry-After header value.
*ร้องขอบ่อยเกินไป รอตามเวลาใน Retry-After header* |
| **AI_UNAVAILABLE** | **503** | All AI model providers are currently unreachable.
*ไม่สามารถเชื่อมต่อ AI model provider ได้ในขณะนี้* |
| **INTERNAL_ERROR** | **500** | Unexpected server error. Please retry.
*เกิดข้อผิดพลาดภายใน server กรุณาลองใหม่* |

**§1 Projects API**

API สำหรับจัดการโปรเจกต์

---

| **GET** | **/api/projects** — ดึงรายการโปรเจกต์ทั้งหมด |
| --- | --- |

| **EN**
Returns all projects that belong to the currently authenticated user, ordered by most recently updated. | **TH**
คืนค่าโปรเจกต์ทั้งหมดที่เป็นของผู้ใช้ที่ล็อกอินอยู่ เรียงตามอัปเดตล่าสุด |
| --- | --- |

▸ **Request Body** *request body*

*No request body required / ไม่ต้องส่ง request body*

---

▸ **Response 200 — data: { projects }** *Response 200 — data: { projects }*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.projects** | Project[] | — | Array of project objects.
*อาร์เรย์ของ project objects* |
| **data.projects[].id** | string (cuid) | — | Unique project identifier.
*รหัสโปรเจกต์ที่ไม่ซ้ำ* |
| **data.projects[].title** | string | — | Project display name.
*ชื่อโปรเจกต์* |
| **data.projects[].srcLang** | Lang (enum) | — | Source language code (JA/ZH/KO/EN/VI).
*รหัสภาษาต้นทาง* |
| **data.projects[].tgtLang** | Lang (enum) | — | Target language code (TH/EN/…).
*รหัสภาษาเป้าหมาย* |
| **data.projects[].style** | TranslationStyle | — | Default translation style for all chapters.
*รูปแบบการแปลเริ่มต้นสำหรับทุก chapter* |
| **data.projects[]._count.chapters** | number | — | Total number of chapters in this project.
*จำนวน chapter ทั้งหมดในโปรเจกต์* |
| **data.projects[].createdAt** | ISO 8601 string | — | Creation timestamp.
*วันและเวลาที่สร้าง* |

▸ **Example** *ตัวอย่าง*

// Response

{

"ok": true,

"data": {

"projects": [

{ "id":"clx1abc","title":"One Piece Vol.1","srcLang":"JA","tgtLang":"TH",

"style":"NATURAL","_count":{"chapters":3},"createdAt":"2025-06-01T10:00:00Z" }

]

}

}

---

| **POST** | **/api/projects** — สร้างโปรเจกต์ใหม่ |
| --- | --- |

| **EN**
Create a new project. Optionally upload a character glossary as CSV/JSON to bootstrap character awareness for the AI pipeline. | **TH**
สร้างโปรเจกต์ใหม่ สามารถอัปโหลด character glossary เป็น CSV/JSON เพื่อช่วยให้ AI รู้จักตัวละครตั้งแต่ต้น |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **title** | string | **✓** | 1–120 chars | Display name for the project.
*ชื่อที่แสดงของโปรเจกต์* |
| **srcLang** | Lang | **✓** | JA|ZH|KO|EN|VI | Source language of the manga.
*ภาษาต้นทางของมังงะ* |
| **tgtLang** | Lang | **✓** | TH|EN|… | Target translation language.
*ภาษาเป้าหมายในการแปล* |
| **style** | TranslationStyle | — | NATURAL|LITERAL|FORMAL|CASUAL (default: NATURAL) | Default translation style.
*รูปแบบการแปลเริ่มต้น* |
| **characters** | array | — | max 200 items | Seed character list for the glossary. Each item: { name, gender, particle?, notes? }.
*รายการตัวละครเริ่มต้น แต่ละรายการ: { name, gender, particle?, notes? }* |

▸ **Response 201 — data: { project }** *Response 201 — data: { project }*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.project** | Project | — | Newly created project object (same shape as GET list item).
*Project object ที่สร้างใหม่ (รูปแบบเดียวกับ GET list)* |

| **error code** | **HTTP** | **Message EN / ข้อความ TH** |
| --- | --- | --- |
| **PROJECT_LIMIT_REACHED** | **403** | Free plan is limited to 5 active projects.
*แผน Free จำกัดที่ 5 โปรเจกต์ที่ใช้งานอยู่* |

▸ **Example** *ตัวอย่าง*

// Request

{ "title":"One Piece Vol.1","srcLang":"JA","tgtLang":"TH","style":"NATURAL",

"characters":[{"name":"Luffy","gender":"MALE","particle":"ว่ะ"},

{"name":"Nami","gender":"FEMALE","particle":"นะ"}] }

// Response HTTP 201

{ "ok":true,"data":{"project":{"id":"clx1abc","title":"One Piece Vol.1",...}} }

---

| **PATCH** | **/api/projects/[id]** — แก้ไขข้อมูลโปรเจกต์ |
| --- | --- |

| **EN**
Partially update a project's metadata. Send only the fields you want to change; all others are preserved. | **TH**
แก้ไขข้อมูลโปรเจกต์บางส่วน ส่งเฉพาะ field ที่ต้องการเปลี่ยน field อื่นจะไม่ถูกกระทบ |
| --- | --- |

▸ **Request Body (all fields optional)** *Request Body (ทุก field เป็น optional)*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **title** | string | — | 1–120 chars | New project title.
*ชื่อโปรเจกต์ใหม่* |
| **style** | TranslationStyle | — | enum value | Change default translation style.
*เปลี่ยนรูปแบบการแปลเริ่มต้น* |
| **srcLang** | Lang | — | enum value | Change source language.
*เปลี่ยนภาษาต้นทาง* |
| **tgtLang** | Lang | — | enum value | Change target language.
*เปลี่ยนภาษาเป้าหมาย* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.project** | Project | — | Updated project object.
*Project object ที่อัปเดตแล้ว* |

| **error code** | **HTTP** | **Message EN / ข้อความ TH** |
| --- | --- | --- |
| **NOT_FOUND** | **404** | Project with this id does not exist or belongs to another user.
*ไม่พบโปรเจกต์หรือเป็นของผู้ใช้อื่น* |

| **DELETE** | **/api/projects/[id]** — ลบโปรเจกต์ |
| --- | --- |

| **EN**
Permanently delete a project. Cascades to all chapters, pages, text regions, and translation jobs. Also deletes all associated Uploadcare files. | **TH**
ลบโปรเจกต์อย่างถาวร จะลบ chapter, page, text region, translation job ที่เกี่ยวข้องทั้งหมด รวมถึงไฟล์บน Uploadcare |
| --- | --- |

▸ **Request Body** *request body*

*No request body required / ไม่ต้องส่ง request body*

---

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **ok** | boolean | — | Always true on success. No data payload.
*true เสมอเมื่อสำเร็จ ไม่มี data payload* |

| **error code** | **HTTP** | **Message EN / ข้อความ TH** |
| --- | --- | --- |
| **NOT_FOUND** | **404** | Project not found.
*ไม่พบโปรเจกต์* |

**§2 Chapters API**

API สำหรับจัดการ Chapter

---

| **POST** | **/api/chapters** — สร้าง Chapter ใหม่ |
| --- | --- |

| **EN**
Create a new chapter under a project. After creation, call POST /api/pages for each uploaded page image. | **TH**
สร้าง chapter ใหม่ในโปรเจกต์ หลังจากสร้างแล้วให้เรียก POST /api/pages สำหรับแต่ละหน้าที่อัปโหลด |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **projectId** | string (cuid) | **✓** | exists, user owns it | Parent project ID.
*ID ของโปรเจกต์ที่เป็นเจ้าของ* |
| **number** | integer | **✓** | 1–9999, unique in project | Chapter number.
*เลข chapter ต้องไม่ซ้ำในโปรเจกต์* |
| **title** | string | **✓** | 1–120 chars | Chapter title (e.g., 'The Beginning').
*ชื่อ chapter เช่น 'จุดเริ่มต้น'* |

▸ **Response 201** *Response 201*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.chapter.id** | integer | — | Chapter numeric ID.
*ID ของ chapter เป็นตัวเลข* |
| **data.chapter.number** | integer | — | Chapter number.
*เลข chapter* |
| **data.chapter.title** | string | — | Chapter title.
*ชื่อ chapter* |
| **data.chapter.status** | PageStatus | — | Always PENDING on creation.
*PENDING เสมอเมื่อเพิ่งสร้าง* |
| **data.chapter.projectId** | string | — | Parent project ID.
*ID ของโปรเจกต์ที่เป็นเจ้าของ* |

| **error code** | **HTTP** | **Message EN / ข้อความ TH** |
| --- | --- | --- |
| **NOT_FOUND** | **404** | Project not found.
*ไม่พบโปรเจกต์* |
| **CHAPTER_EXISTS** | **409** | A chapter with this number already exists in the project.
*มี chapter หมายเลขนี้อยู่แล้วในโปรเจกต์* |

| **PATCH** | **/api/chapters/[id]** — แก้ไข Chapter |
| --- | --- |

| **EN**
Update a chapter's title, number, or status. Status can only be advanced forward (PENDING→TRANSLATED→APPROVED), not rolled back via this endpoint. | **TH**
แก้ไขชื่อ, เลข, หรือสถานะของ chapter สถานะเดินหน้าเท่านั้น (PENDING→TRANSLATED→APPROVED) ไม่สามารถย้อนกลับผ่าน endpoint นี้ |
| --- | --- |

▸ **Request Body (all optional)** *Request Body (ทุก field เป็น optional)*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **title** | string | — | 1–120 chars | New chapter title.
*ชื่อ chapter ใหม่* |
| **number** | integer | — | 1–9999, unique | New chapter number.
*เลข chapter ใหม่* |
| **status** | PageStatus | — | forward transitions only | New chapter status.
*สถานะ chapter ใหม่* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.chapter** | Chapter | — | Updated chapter object.
*Chapter object ที่อัปเดตแล้ว* |

| **DELETE** | **/api/chapters/[id]** — ลบ Chapter |
| --- | --- |

| **EN**
Permanently delete a chapter and all its pages, text regions, and job records. Uploadcare files are also deleted. | **TH**
ลบ chapter และ page, text region, job ที่เกี่ยวข้องอย่างถาวร รวมถึงไฟล์บน Uploadcare |
| --- | --- |

▸ **Request Body** *request body*

*No request body required / ไม่ต้องส่ง request body*

---

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **ok** | boolean | — | true on success.
*true เมื่อสำเร็จ* |

**§3 Pages API**

API สำหรับจัดการ Page

---

| **POST** | **/api/pages** — เพิ่มหน้าใหม่ |
| --- | --- |

| **EN**
Register a page after its image has been uploaded to Uploadcare. The Uploadcare file ID is stored (not the full URL — see DB schema §D). Call this once per page image. | **TH**
ลงทะเบียน page หลังจากอัปโหลดรูปภาพไปยัง Uploadcare เรียกครั้งเดียวต่อ page image โดยเก็บแค่ Uploadcare file ID ไม่ใช่ URL เต็ม |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **chapterId** | integer | **✓** | exists, user owns it | Parent chapter ID.
*ID ของ chapter* |
| **pageNumber** | integer | **✓** | 1–9999, unique in chapter | Page number within chapter.
*เลขหน้าใน chapter ต้องไม่ซ้ำ* |
| **srcFileId** | string (UUID) | **✓** | valid Uploadcare UUID | Uploadcare file UUID for the original raw scan.
*UUID ของไฟล์ใน Uploadcare สำหรับสแกนต้นฉบับ* |

▸ **Response 201** *Response 201*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.page.id** | integer | — | Page numeric ID.
*ID ของ page* |
| **data.page.pageNumber** | integer | — | Page number.
*เลขหน้า* |
| **data.page.srcFileId** | string | — | Uploadcare UUID for original image.
*UUID ของรูปต้นฉบับบน Uploadcare* |
| **data.page.cleanFileId** | string | ✓ | Uploadcare UUID of inpainted image. Null until inpainting runs.
*UUID ของรูปหลัง inpainting ว่างจนกว่าจะ inpaint* |
| **data.page.renderedFileId** | string | ✓ | Uploadcare UUID of final composite. Null until render runs.
*UUID ของรูปสุดท้ายที่ render แล้ว ว่างจนกว่าจะ render* |
| **data.page.status** | PageStatus | — | Always PENDING on creation.
*PENDING เสมอเมื่อเพิ่งเพิ่ม* |
| **data.page.chapterId** | integer | — | Parent chapter ID.
*ID ของ chapter* |

| **error code** | **HTTP** | **Message EN / ข้อความ TH** |
| --- | --- | --- |
| **NOT_FOUND** | **404** | Chapter not found.
*ไม่พบ chapter* |
| **PAGE_EXISTS** | **409** | Page with this number already exists in chapter.
*มีหน้านี้อยู่แล้วใน chapter* |
| **STORAGE_LIMIT** | **403** | Storage quota exceeded. Delete old files or upgrade plan.
*พื้นที่จัดเก็บเต็ม ลบไฟล์เก่าหรืออัปเกรดแผน* |

| **GET** | **/api/pages/[id]** — ดึงข้อมูล Page พร้อม TextRegion |
| --- | --- |

| **EN**
Fetch a single page with all its text regions and the latest translation job status. This is the primary data load call for the editor. | **TH**
ดึง page พร้อม text region ทั้งหมดและสถานะ job ล่าสุด เป็น call หลักที่ editor ใช้ตอนโหลด |
| --- | --- |

▸ **Request Body** *request body*

*No request body required / ไม่ต้องส่ง request body*

---

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.page** | Page | — | The page object.
*Page object* |
| **data.regions** | TextRegion[] | — | All text regions for this page, ordered by bubbleIndex.
*Text region ทั้งหมดของ page เรียงตาม bubbleIndex* |
| **data.regions[].id** | integer | — | Region ID.
*ID ของ region* |
| **data.regions[].bubbleIndex** | integer | — | Reading order index (1 = first bubble).
*ลำดับการอ่าน (1 = bubble แรก)* |
| **data.regions[].regionType** | RegionType | — | DIALOGUE / THOUGHT / NARRATION / SFX / SIGNAGE.
*ประเภทของ region* |
| **data.regions[].bboxX** | integer | — | Bounding box x-coordinate in original image pixels.
*พิกัด x ในภาพต้นฉบับ (pixel)* |
| **data.regions[].bboxY** | integer | — | Bounding box y-coordinate.
*พิกัด y ในภาพต้นฉบับ (pixel)* |
| **data.regions[].bboxW** | integer | — | Bounding box width.
*ความกว้างของ bounding box* |
| **data.regions[].bboxH** | integer | — | Bounding box height.
*ความสูงของ bounding box* |
| **data.regions[].originalText** | string | — | OCR-extracted source text.
*ข้อความต้นฉบับที่ OCR ดึงออกมา* |
| **data.regions[].translatedText** | string | ✓ | AI-suggested translation. Null before translation step.
*คำแปลที่ AI แนะนำ ว่างก่อน translation step* |
| **data.regions[].speakerName** | string | ✓ | Detected speaker name. Null if not identified.
*ชื่อตัวละครที่พูด ว่างหากไม่ระบุ* |
| **data.regions[].speakerGender** | GenderCode | — | MALE / FEMALE / NEUTRAL / UNKNOWN.
*เพศของตัวละคร* |
| **data.regions[].thaiParticle** | string | ✓ | Suggested Thai sentence-final particle (ครับ/ค่ะ/…).
*คำลงท้ายภาษาไทยที่แนะนำ (ครับ/ค่ะ/…)* |
| **data.regions[].confidence** | integer | — | AI confidence 0–100.
*ความมั่นใจของ AI 0–100* |
| **data.regions[].isApproved** | boolean | — | Whether a human has approved this translation.
*มนุษย์อนุมัติคำแปลนี้แล้วหรือไม่* |
| **data.regions[].isManual** | boolean | — | True if region was drawn manually (not by AI).
*true หาก region วาดด้วยมือ ไม่ใช่ AI* |
| **data.regions[].fontSizeOverride** | integer | ✓ | Manual font size px. Null = auto-fit.
*ขนาดฟอนต์ที่กำหนดเอง ว่าง = ปรับอัตโนมัติ* |
| **data.job** | TranslationJob | ✓ | Latest translation job for this page. Null if never run.
*Translation job ล่าสุดของ page นี้ ว่างหากยังไม่เคยรัน* |
| **data.job.status** | JobStatus | — | QUEUED / RUNNING / DONE / ERROR.
*สถานะของ job* |
| **data.job.step** | integer | — | Last completed pipeline step: 0=none, 1=vision, 2=chars, 3=translated.
*step ล่าสุดที่เสร็จแล้ว: 0=ยังไม่มี 1=vision 2=chars 3=แปลแล้ว* |

| **DELETE** | **/api/pages/[id]** — ลบ Page |
| --- | --- |

| **EN**
Delete a page and all its text regions. Also calls the Uploadcare REST API to delete the original, cleaned, and rendered image files to free storage. | **TH**
ลบ page และ text region ทั้งหมด รวมถึงเรียก Uploadcare REST API เพื่อลบไฟล์รูปต้นฉบับ cleaned และ rendered เพื่อประหยัดพื้นที่ |
| --- | --- |

▸ **Request Body** *request body*

*No request body required / ไม่ต้องส่ง request body*

---

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **ok** | boolean | — | true on success.
*true เมื่อสำเร็จ* |

**§4 Translation Jobs API**

API สำหรับจัดการ Translation Job

---

| **POST** | **/api/jobs/process** — เพิ่ม Job แปลสำหรับ 1 Page |
| --- | --- |

| **EN**
Queue a full 3-step AI translation job for a single page. Returns immediately with a job ID. The actual work is done asynchronously by the cron worker. Poll GET /api/jobs/[id] for status. | **TH**
เพิ่ม AI translation job 3 ขั้นตอนสำหรับ page เดียว คืนค่า job ID ทันที งานจริงทำแบบ async โดย cron worker ใช้ GET /api/jobs/[id] ตรวจสอบสถานะ |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **pageId** | integer | **✓** | exists, user owns it | Target page to process.
*Page ที่ต้องการแปล* |
| **modelOverride** | string | — | valid OpenRouter model ID | Override the default AI model (requires user-provided API key).
*ระบุ AI model ที่ต้องการ (ต้องมี API key ของตัวเอง)* |
| **forceRerun** | boolean | — | default false | Re-queue even if a completed job exists.
*เพิ่ม job ใหม่แม้ว่าจะมี job ที่เสร็จแล้ว* |

▸ **Response 202** *Response 202*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.jobId** | integer | — | ID of the newly created TranslationJob.
*ID ของ TranslationJob ที่สร้างใหม่* |
| **data.status** | string | — | Always 'QUEUED' immediately after creation.
*QUEUED เสมอทันทีหลังสร้าง* |
| **data.position** | integer | — | Estimated queue position (1 = next to run).
*ตำแหน่งในคิว (1 = จะรันต่อไป)* |

| **error code** | **HTTP** | **Message EN / ข้อความ TH** |
| --- | --- | --- |
| **NOT_FOUND** | **404** | Page not found.
*ไม่พบ page* |
| **JOB_RUNNING** | **409** | A job for this page is already queued or running.
*มี job ของ page นี้กำลังรออยู่หรือรันอยู่แล้ว* |

| **POST** | **/api/jobs/process-batch** — เพิ่ม Job แปลสำหรับทุก Page ใน Chapter |
| --- | --- |

| **EN**
Queue translation jobs for all PENDING pages in a chapter in a single call. Already-processed pages are skipped unless forceRerun is true. | **TH**
เพิ่ม translation job สำหรับทุก page ที่มีสถานะ PENDING ใน chapter ใน call เดียว page ที่แปลแล้วจะถูกข้ามเว้นแต่ forceRerun เป็น true |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **chapterId** | integer | **✓** | exists, user owns it | Target chapter.
*Chapter ที่ต้องการแปล* |
| **forceRerun** | boolean | — | default false | Re-queue all pages including completed ones.
*เพิ่ม job ใหม่สำหรับทุก page รวมถึงที่แปลแล้ว* |

▸ **Response 202** *Response 202*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.queued** | integer | — | Number of new jobs successfully added to the queue.
*จำนวน job ใหม่ที่เพิ่มในคิวสำเร็จ* |
| **data.skipped** | integer | — | Number of pages skipped (already processed, no forceRerun).
*จำนวน page ที่ข้ามเพราะแปลแล้ว* |
| **data.jobIds** | integer[] | — | IDs of all newly created jobs.
*ID ของ job ที่สร้างใหม่ทั้งหมด* |

| **GET** | **/api/jobs/[id]** — ตรวจสอบสถานะ Job |
| --- | --- |

| **EN**
Poll for the current status of a translation job. The editor uses this on a 3-second interval to show real-time progress during processing. | **TH**
ตรวจสอบสถานะปัจจุบันของ translation job editor ใช้ call นี้ทุก 3 วินาทีเพื่อแสดงความคืบหน้าแบบ real-time |
| --- | --- |

▸ **Request Body** *request body*

*No request body required / ไม่ต้องส่ง request body*

---

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.job.id** | integer | — | Job ID.
*ID ของ job* |
| **data.job.status** | JobStatus | — | QUEUED / RUNNING / DONE / ERROR.
*สถานะ job* |
| **data.job.step** | integer | — | 0=pending 1=vision done 2=chars done 3=translation done.
*ขั้นตอนล่าสุดที่เสร็จ 0=รอ 1=vision 2=chars 3=แปล* |
| **data.job.stepLabel** | string | — | Human-readable step description (EN).
*คำอธิบาย step ที่อ่านได้ (EN)* |
| **data.job.stepLabelTH** | string | — | Human-readable step description (TH).
*คำอธิบาย step ที่อ่านได้ (TH)* |
| **data.job.errorMessage** | string | ✓ | Error detail if status is ERROR.
*รายละเอียดข้อผิดพลาดถ้าสถานะเป็น ERROR* |
| **data.job.startedAt** | ISO 8601 | ✓ | When the cron worker started processing.
*เวลาที่ cron worker เริ่มทำงาน* |
| **data.job.completedAt** | ISO 8601 | ✓ | When the job finished (done or error).
*เวลาที่ job เสร็จหรือเกิด error* |

▸ **Example** *ตัวอย่าง*

// Response — job in progress at vision step

{ "ok":true,"data":{"job":{

"id":42,"status":"RUNNING","step":1,

"stepLabel":"Detecting speech bubbles…",

"stepLabelTH":"กำลังตรวจหา speech bubble…",

"errorMessage":null,"startedAt":"2025-06-01T10:01:00Z","completedAt":null

}}}

---

**§5 AI Pipeline — Internal Endpoints**

AI Pipeline — Endpoint ภายใน (เรียกโดย Cron เท่านั้น)

---

**⚠️ Internal use only.** These three endpoints are called sequentially by the cron worker (/api/cron/run-jobs). They must not be called directly from the browser. They are protected by a server-to-server secret header (X-Internal-Secret).

**⚠️ สำหรับใช้ภายในเท่านั้น.** endpoint ทั้งสามนี้ถูกเรียกตามลำดับโดย cron worker (/api/cron/run-jobs) ห้ามเรียกโดยตรงจาก browser ป้องกันด้วย header X-Internal-Secret

| **POST** | **/api/ai/vision** — ตรวจจับ Speech Bubble (OCR) |
| --- | --- |

| **EN**
Step 1 of the AI pipeline. Sends the page image to GLM-4V (or configured vision model) and returns bounding boxes + raw OCR text for every detected text region. | **TH**
ขั้นตอนที่ 1 ของ AI pipeline ส่งรูปหน้าให้ GLM-4V (หรือ vision model ที่ตั้งค่า) คืนค่า bounding box และข้อความ OCR ของแต่ละ region |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **pageId** | integer | **✓** | valid page | Target page ID.
*ID ของ page เป้าหมาย* |
| **imageUrl** | string | **✓** | full Uploadcare CDN URL | Public URL of the page image.
*URL สาธารณะของรูปหน้า* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.regions** | RawRegion[] | — | Detected regions from the vision model.
*Region ที่ตรวจพบจาก vision model* |
| **data.regions[].bubbleIndex** | integer | — | Reading-order index assigned by the model.
*ลำดับการอ่านที่ model กำหนด* |
| **data.regions[].regionType** | RegionType | — | DIALOGUE / THOUGHT / NARRATION / SFX / SIGNAGE.
*ประเภท region* |
| **data.regions[].bbox** | object | — | { x, y, w, h } in original image pixels.
*{ x, y, w, h } หน่วยเป็น pixel ของภาพต้นฉบับ* |
| **data.regions[].ocrText** | string | — | Raw text extracted by OCR.
*ข้อความดิบที่ OCR ดึงออกมา* |
| **data.savedCount** | integer | — | Number of TextRegion rows written to DB.
*จำนวนแถว TextRegion ที่บันทึกลง DB* |

| **POST** | **/api/ai/characters** — วิเคราะห์ตัวละคร & เพศ |
| --- | --- |

| **EN**
Step 2. Sends page image + list of detected bubble bounding boxes to the Character ID agent. Returns speaker identity, gender presentation, and Thai polite particle suggestion per bubble. | **TH**
ขั้นตอนที่ 2 ส่งรูปหน้า + bounding box ของ bubble ให้ Character ID agent คืนค่าชื่อตัวละคร เพศ และคำลงท้ายภาษาไทยที่แนะนำต่อ bubble |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **pageId** | integer | **✓** | valid page | Target page ID.
*ID ของ page* |
| **imageUrl** | string | **✓** | CDN URL | Page image URL.
*URL ของรูปหน้า* |
| **regions** | RawRegion[] | **✓** | from vision step | Regions returned by the vision step.
*Region จาก vision step* |
| **glossary** | object[] | — | project's characters | Project character list for name-matching.
*รายการตัวละครของโปรเจกต์สำหรับจับคู่ชื่อ* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.speakers** | SpeakerMeta[] | — | One entry per region.
*หนึ่งรายการต่อ region* |
| **data.speakers[].regionId** | integer | — | Matching TextRegion ID.
*ID ของ TextRegion ที่ตรงกัน* |
| **data.speakers[].speakerName** | string | ✓ | Matched character name or null if unknown.
*ชื่อตัวละครที่จับคู่ได้ หรือ null ถ้าไม่รู้จัก* |
| **data.speakers[].gender** | GenderCode | — | MALE / FEMALE / NEUTRAL / UNKNOWN.
*เพศของตัวละคร* |
| **data.speakers[].confidence** | integer | — | Confidence 0–100 for gender detection.
*ความมั่นใจ 0–100 ในการตรวจจับเพศ* |
| **data.speakers[].thaiParticle** | string | ✓ | Suggested Thai particle (ครับ / ค่ะ / นะ / …).
*คำลงท้ายภาษาไทยที่แนะนำ (ครับ / ค่ะ / นะ / …)* |

| **POST** | **/api/ai/translate** — แปลข้อความทุก Region |
| --- | --- |

| **EN**
Step 3. Sends all regions with speaker metadata to the Translation agent. Returns translated text per region, incorporating the project glossary and cross-chapter translation memory for consistency. | **TH**
ขั้นตอนที่ 3 ส่ง region พร้อม speaker metadata ให้ Translation agent คืนค่าคำแปลต่อ region โดยใช้ glossary และ translation memory ข้าม chapter เพื่อความสม่ำเสมอ |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **pageId** | integer | **✓** | valid page | Target page ID.
*ID ของ page* |
| **regions** | RawRegion[] | **✓** | with ocrText | Regions with OCR text.
*Region พร้อมข้อความ OCR* |
| **speakers** | SpeakerMeta[] | **✓** | from chars step | Speaker metadata from step 2.
*Speaker metadata จาก step 2* |
| **glossary** | object[] | — | project chars | Project character glossary.
*Glossary ของโปรเจกต์* |
| **memory** | object[] | — | recent approved translations | Translation memory: [{ sourceText, targetText }].
*Translation memory: [{ sourceText, targetText }]* |
| **style** | TranslationStyle | — | project default | Override translation style for this run.
*ระบุ translation style สำหรับ run นี้* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.translations** | Translation[] | — | One per region.
*หนึ่งรายการต่อ region* |
| **data.translations[].regionId** | integer | — | TextRegion ID.
*ID ของ TextRegion* |
| **data.translations[].translatedText** | string | — | Translated text with correct Thai particle.
*คำแปลพร้อมคำลงท้ายภาษาไทยที่ถูกต้อง* |
| **data.translations[].confidence** | integer | — | Translation confidence 0–100.
*ความมั่นใจในการแปล 0–100* |
| **data.memoryHits** | integer | — | Number of regions matched from translation memory.
*จำนวน region ที่ตรงกับ translation memory* |

**§6 Text Regions API**

API สำหรับจัดการ Text Region

---

| **PATCH** | **/api/regions/[id]** — แก้ไข Text Region |
| --- | --- |

| **EN**
Update a single text region. Called every time a translator edits a cell in the spreadsheet panel. Only send the fields that changed; others are untouched. | **TH**
แก้ไข text region เดียว เรียกทุกครั้งที่นักแปลแก้ไข cell ใน spreadsheet ส่งเฉพาะ field ที่เปลี่ยน field อื่นไม่ถูกกระทบ |
| --- | --- |

▸ **Request Body (all optional)** *Request Body (ทุก field optional)*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **translatedText** | string | — | max 2000 chars | Updated translation text.
*ข้อความแปลที่แก้ไข* |
| **originalText** | string | — | max 2000 chars | Corrected OCR source text.
*ข้อความต้นฉบับที่แก้ไข OCR* |
| **isApproved** | boolean | — | true | false | Mark translation as approved or revoke approval.
*ทำเครื่องหมายอนุมัติหรือยกเลิกการอนุมัติ* |
| **speakerGender** | GenderCode | — | enum value | Override detected gender.
*แก้ไขเพศที่ตรวจจับได้* |
| **thaiParticle** | string | — | max 10 chars | Override Thai sentence-final particle.
*แก้ไขคำลงท้ายภาษาไทย* |
| **speakerName** | string | — | max 80 chars | Override detected speaker name.
*แก้ไขชื่อตัวละครที่พูด* |
| **fontSizeOverride** | integer | — | 8–72 px | Manual font size for the rendered output. Null to auto-fit.
*ขนาดฟอนต์ในการ render ว่าง = ปรับอัตโนมัติ* |
| **bboxX** | integer | — | ≥0 | Update bounding box x after manual resize on canvas.
*อัปเดต x หลังปรับ bounding box บน canvas* |
| **bboxY** | integer | — | ≥0 | Update bounding box y.
*อัปเดต y* |
| **bboxW** | integer | — | ≥10 | Update bounding box width.
*อัปเดตความกว้าง* |
| **bboxH** | integer | — | ≥10 | Update bounding box height.
*อัปเดตความสูง* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.region** | TextRegion | — | Full updated TextRegion object.
*TextRegion object ที่อัปเดตครบทุก field* |

| **POST** | **/api/regions/bulk-approve** — อนุมัติ Region ทีเดียวหลายรายการ |
| --- | --- |

| **EN**
Set isApproved = true on all text regions in a page that have a confidence score at or above the given threshold. Returns the count of newly approved regions. | **TH**
ตั้งค่า isApproved = true สำหรับ text region ทุก region ใน page ที่มี confidence ≥ threshold ที่กำหนด คืนค่าจำนวน region ที่อนุมัติใหม่ |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **pageId** | integer | **✓** | valid page | Target page.
*Page เป้าหมาย* |
| **minConfidence** | integer | — | 0–100, default 75 | Only approve regions with confidence ≥ this value.
*อนุมัติเฉพาะ region ที่ confidence ≥ ค่านี้* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.approved** | integer | — | Count of regions newly set to approved.
*จำนวน region ที่อนุมัติใหม่* |
| **data.skipped** | integer | — | Count of regions skipped (below threshold or already approved).
*จำนวน region ที่ข้ามเพราะ confidence ต่ำหรืออนุมัติแล้ว* |

| **POST** | **/api/regions/manual** — สร้าง Region ด้วยมือ |
| --- | --- |

| **EN**
Create a new text region that was manually drawn by the translator on the canvas. Optionally trigger an immediate single-region translation. | **TH**
สร้าง text region ใหม่ที่นักแปลวาดด้วยมือบน canvas สามารถเลือกให้แปลทันทีสำหรับ region เดียวนี้ได้ |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **pageId** | integer | **✓** | valid page | Target page.
*Page เป้าหมาย* |
| **bboxX** | integer | **✓** | ≥0 | Bounding box x in original image pixels.
*พิกัด x ใน pixel ของภาพต้นฉบับ* |
| **bboxY** | integer | **✓** | ≥0 | Bounding box y.
*พิกัด y* |
| **bboxW** | integer | **✓** | ≥10 | Bounding box width.
*ความกว้าง* |
| **bboxH** | integer | **✓** | ≥10 | Bounding box height.
*ความสูง* |
| **originalText** | string | **✓** | 1–2000 chars | Source text typed by the translator.
*ข้อความต้นฉบับที่นักแปลพิมพ์* |
| **regionType** | RegionType | — | default DIALOGUE | Region type.
*ประเภทของ region* |
| **translate** | boolean | — | default false | If true, immediately translate this region using the AI pipeline (synchronous, up to 5 s).
*ถ้า true ให้แปลทันทีใช้ AI pipeline (synchronous ใช้เวลาสูงสุด 5 วินาที)* |

▸ **Response 201** *Response 201*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.region** | TextRegion | — | Newly created region. If translate=true, translatedText is populated.
*Region ที่สร้างใหม่ ถ้า translate=true จะมี translatedText* |

| **DELETE** | **/api/regions/[id]** — ลบ Text Region |
| --- | --- |

| **EN**
Delete a single text region. Used when the AI detected a false positive (a region that is not actually a speech bubble). | **TH**
ลบ text region เดียว ใช้เมื่อ AI ตรวจพบ false positive (region ที่ไม่ใช่ speech bubble จริงๆ) |
| --- | --- |

▸ **Request Body** *request body*

*No request body required / ไม่ต้องส่ง request body*

---

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **ok** | boolean | — | true on success.
*true เมื่อสำเร็จ* |

**§7 Render API**

API สำหรับ Render ภาพสุดท้าย

---

| **POST** | **/api/render/page** — Render ภาพสุดท้ายสำหรับ 1 Page |
| --- | --- |

| **EN**
Compose the final output image for a single page: (1) run inpainting to remove original text, (2) overlay translated Thai text into each bubble with auto-fitted font size, (3) upload result to Uploadcare and store the file ID. | **TH**
สร้างภาพสุดท้ายสำหรับ page เดียว: (1) inpaint เพื่อลบข้อความต้นฉบับ (2) วางคำแปลภาษาไทยลงใน bubble พร้อมปรับขนาดฟอนต์อัตโนมัติ (3) อัปโหลดผลลัพธ์ขึ้น Uploadcare |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **pageId** | integer | **✓** | valid page, all regions approved | Target page. Should have all regions approved before rendering.
*Page เป้าหมาย ควรอนุมัติ region ทั้งหมดก่อน render* |
| **fontFamily** | string | — | default 'Noto Sans Thai' | Font to use for Thai text rendering.
*ฟอนต์ที่ใช้สำหรับแสดงผลข้อความไทย* |
| **lineHeightPx** | integer | — | default auto | Override line height in pixels.
*ระบุ line height หน่วยเป็น pixel* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.renderedFileId** | string (UUID) | — | Uploadcare UUID of the composed final image.
*UUID ของรูปสุดท้ายที่ compose แล้วบน Uploadcare* |
| **data.renderedUrl** | string | — | Full CDN URL reconstructed from the file ID.
*URL เต็มของ CDN ที่สร้างจาก file ID* |

| **error code** | **HTTP** | **Message EN / ข้อความ TH** |
| --- | --- | --- |
| **REGIONS_NOT_APPROVED** | **422** | Not all text regions on this page are approved. Approve or delete unapproved regions first.
*ยังมี text region ที่ยังไม่อนุมัติในหน้านี้ อนุมัติหรือลบก่อน render* |
| **INPAINT_FAILED** | **503** | Inpainting service is unreachable.
*ไม่สามารถเชื่อมต่อ inpainting service ได้* |

| **POST** | **/api/render/chapter** — Render ทุก Page ใน Chapter |
| --- | --- |

| **EN**
Sequentially render all approved pages in a chapter. Pages that are not yet fully approved are skipped. Returns counts of rendered and skipped pages. | **TH**
Render ทุก page ที่อนุมัติแล้วใน chapter ตามลำดับ page ที่ยังไม่อนุมัติครบจะถูกข้าม คืนค่าจำนวน page ที่ render และที่ข้าม |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **chapterId** | integer | **✓** | valid chapter | Target chapter.
*Chapter เป้าหมาย* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.rendered** | integer | — | Number of pages successfully rendered.
*จำนวน page ที่ render สำเร็จ* |
| **data.skipped** | integer | — | Number of pages skipped (unapproved regions).
*จำนวน page ที่ข้าม (ยังมี region ไม่อนุมัติ)* |
| **data.errors** | object[] | — | [{ pageId, error }] for any pages that failed.
*[{ pageId, error }] สำหรับ page ที่เกิดข้อผิดพลาด* |

**§8 Export API**

API สำหรับ Export ไฟล์

---

| **POST** | **/api/export/zip** — Export เป็น ZIP (PNG) |
| --- | --- |

| **EN**
Package all rendered page images for a chapter into a ZIP archive and upload it to Uploadcare. Returns a time-limited signed download URL valid for 24 hours. | **TH**
รวมรูป page ที่ render แล้วทั้งหมดของ chapter เป็น ZIP แล้วอัปโหลดไปยัง Uploadcare คืนค่า signed download URL ที่ใช้ได้ 24 ชั่วโมง |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **chapterId** | integer | **✓** | valid chapter | Target chapter.
*Chapter เป้าหมาย* |
| **includeOriginal** | boolean | — | default false | Include original (un-cleaned) page scans alongside rendered pages.
*รวมสแกนต้นฉบับด้วยนอกเหนือจาก page ที่ render แล้ว* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.downloadUrl** | string | — | Signed CDN URL for the ZIP file, valid for 24 h.
*Signed CDN URL สำหรับดาวน์โหลด ZIP ใช้ได้ 24 ชั่วโมง* |
| **data.pageCount** | integer | — | Number of pages included in the archive.
*จำนวน page ใน ZIP* |
| **data.expiresAt** | ISO 8601 | — | URL expiry timestamp.
*เวลาหมดอายุของ URL* |

| **error code** | **HTTP** | **Message EN / ข้อความ TH** |
| --- | --- | --- |
| **NO_RENDERED_PAGES** | **422** | No rendered pages found. Run POST /api/render/chapter first.
*ไม่พบ page ที่ render แล้ว กรุณา render ก่อน export* |

| **POST** | **/api/export/pdf** — Export เป็น PDF |
| --- | --- |

| **EN**
Combine all rendered page images into a single multi-page PDF using pdf-lib. Upload to Uploadcare and return a signed download URL valid for 24 hours. | **TH**
รวมรูป page ที่ render แล้วทั้งหมดเป็น PDF หลายหน้าโดยใช้ pdf-lib อัปโหลดไป Uploadcare และคืนค่า signed URL ที่ใช้ได้ 24 ชั่วโมง |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **chapterId** | integer | **✓** | valid chapter | Target chapter.
*Chapter เป้าหมาย* |
| **includeTitle** | boolean | — | default true | Add a title page with project/chapter name as the first PDF page.
*เพิ่มหน้าปกที่มีชื่อโปรเจกต์และ chapter เป็นหน้าแรกของ PDF* |
| **quality** | string | — | HIGH|MEDIUM|LOW, default HIGH | Image compression quality. Lower = smaller file size.
*คุณภาพการบีบอัดรูป ต่ำ = ไฟล์เล็กลง* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.downloadUrl** | string | — | Signed URL for the PDF, valid 24 h.
*Signed URL สำหรับดาวน์โหลด PDF ใช้ได้ 24 ชั่วโมง* |
| **data.pageCount** | integer | — | Number of pages in the PDF.
*จำนวนหน้าใน PDF* |
| **data.fileSizeKb** | integer | — | Approximate PDF file size in kilobytes.
*ขนาดไฟล์ PDF โดยประมาณ (KB)* |
| **data.expiresAt** | ISO 8601 | — | URL expiry timestamp.
*เวลาหมดอายุของ URL* |

**§9 Storage API**

API สำหรับจัดการไฟล์บน Uploadcare

---

| **POST** | **/api/storage/sign** — สร้าง Signed Upload URL |
| --- | --- |

| **EN**
Generate a short-lived signed Uploadcare upload URL server-side. The browser calls this first, then uploads the file directly to Uploadcare using the returned credentials. This keeps the Uploadcare secret key on the server only. | **TH**
สร้าง signed Uploadcare upload URL ฝั่ง server browser เรียก endpoint นี้ก่อน จากนั้นอัปโหลดไฟล์ตรงไปยัง Uploadcare โดยใช้ credentials ที่ได้คืนมา เพื่อให้ secret key อยู่บน server เท่านั้น |
| --- | --- |

▸ **Request Body** *request body*

| **Field** | **Type** | **Req** | **Validation** | **Description / คำอธิบาย** |
| --- | --- | --- | --- | --- |
| **fileName** | string | **✓** | max 200 chars | Original file name for metadata.
*ชื่อไฟล์ต้นฉบับสำหรับ metadata* |
| **mimeType** | string | **✓** | image/jpeg|image/png|image/webp | File MIME type.
*MIME type ของไฟล์* |

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.uploadUrl** | string | — | Signed Uploadcare upload endpoint URL.
*URL ของ Uploadcare upload endpoint ที่ signed แล้ว* |
| **data.fileId** | string (UUID) | — | Pre-assigned Uploadcare file UUID. Store this to reference the file after upload.
*UUID ที่กำหนดล่วงหน้า เก็บไว้เพื่ออ้างอิงหลังอัปโหลด* |
| **data.expiresAt** | ISO 8601 | — | Signed URL expiry (typically 30 min from now).
*เวลาหมดอายุของ URL (โดยทั่วไป 30 นาทีนับจากนี้)* |
| **data.maxSizeMb** | integer | — | Maximum allowed file size in megabytes (plan-dependent).
*ขนาดไฟล์สูงสุดที่อนุญาต (ขึ้นกับแผน)* |

▸ **Upload Flow Example** *ตัวอย่าง Upload Flow*

// 1. Get signed URL from server

const { data } = await fetch("/api/storage/sign", {

method:"POST", body: JSON.stringify({ fileName:"page01.jpg", mimeType:"image/jpeg" }) }).then(r=>r.json());

// 2. Upload file directly to Uploadcare (client-side)

const form = new FormData();

form.append("UPLOADCARE_PUB_KEY", PUBLIC_KEY);

form.append("UPLOADCARE_STORE", "1");

form.append("file", fileBlob);

await fetch(data.uploadUrl, { method:"POST", body: form });

// 3. Register page in our DB using the pre-assigned fileId

await fetch("/api/pages", {

method:"POST", body: JSON.stringify({ chapterId, pageNumber:1, srcFileId: data.fileId }) });

---

| **DELETE** | **/api/storage/[fileId]** — ลบไฟล์จาก Uploadcare |
| --- | --- |

| **EN**
Delete a file from Uploadcare storage by its UUID. Called automatically when a page is deleted. Can also be called directly to clean up orphaned files. | **TH**
ลบไฟล์จาก Uploadcare storage ด้วย UUID ถูกเรียกอัตโนมัติเมื่อลบ page สามารถเรียกโดยตรงเพื่อทำความสะอาดไฟล์ที่ไม่มีเจ้าของ |
| --- | --- |

▸ **Request Body** *request body*

*No request body required / ไม่ต้องส่ง request body*

---

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **ok** | boolean | — | true even if the file did not exist on Uploadcare (idempotent).
*true แม้ไฟล์ไม่มีอยู่บน Uploadcare (idempotent)* |

**§10 Cron Worker Endpoint**

Endpoint สำหรับ Cron Worker

---

| **GET** | **/api/cron/run-jobs** — Cron Worker — รัน Job ในคิว |
| --- | --- |

| **EN**
Called by Vercel Cron every 30 seconds. Picks the oldest QUEUED TranslationJob, runs the full 3-step AI pipeline (vision → characters → translate), saves all results, then marks the job DONE. Protected by X-Cron-Secret header. | **TH**
Vercel Cron เรียกทุก 30 วินาที เลือก TranslationJob ที่เก่าที่สุดในสถานะ QUEUED รัน AI pipeline 3 ขั้นตอน (vision → characters → translate) บันทึกผลทั้งหมด แล้วตั้งสถานะ DONE ป้องกันด้วย header X-Cron-Secret |
| --- | --- |

**Required Headers / Header ที่จำเป็น:** X-Cron-Secret: <env CRON_SECRET> (verified server-side; any other value returns 401)

▸ **Request Body** *request body*

*No request body required / ไม่ต้องส่ง request body*

---

▸ **Response 200** *Response 200*

| **Field** | **Type** | **Null?** | **Description / คำอธิบาย** |
| --- | --- | --- | --- |
| **data.processed** | integer | — | Number of jobs processed in this invocation (0 or 1).
*จำนวน job ที่ประมวลผลใน call นี้ (0 หรือ 1)* |
| **data.jobId** | integer | ✓ | ID of the job that was processed. Null if queue was empty.
*ID ของ job ที่ประมวลผล null ถ้าคิวว่าง* |
| **data.status** | JobStatus | ✓ | Final status of the processed job.
*สถานะสุดท้ายของ job ที่ประมวลผล* |
| **data.durationMs** | integer | ✓ | Total time taken for the full pipeline in milliseconds.
*เวลาที่ใช้ทั้งหมดสำหรับ pipeline เต็มรูปแบบ (millisecond)* |

▸ **Full Pipeline Trace Example** *ตัวอย่าง Pipeline ครบทั้งกระบวนการ*

// Cron fires at T+0s

1. SELECT job WHERE status=QUEUED ORDER BY createdAt LIMIT 1 → job #42, page #7

2. UPDATE job SET status=RUNNING, step=0, startedAt=now()

3. POST /api/ai/vision { pageId:7, imageUrl:"https://ucarecdn.com/..." }

← { regions: 14 raw regions }

INSERT 14 TextRegion rows | UPDATE job SET step=1

4. POST /api/ai/characters { pageId:7, imageUrl:..., regions:[...], glossary:[...] }

← { speakers: [{ regionId, speakerName:"Luffy", gender:"MALE", thaiParticle:"ว่ะ" }, ...] }

UPDATE 14 TextRegion rows | UPDATE job SET step=2

5. POST /api/ai/translate { pageId:7, regions:[...], speakers:[...], memory:[...] }

← { translations: [{ regionId, translatedText:"นั่นต้องเป็นคืนของฉัน!ว่ะ", confidence:88 }, ...] }

UPDATE 14 TextRegion rows | UPDATE job SET step=3

6. UPDATE page SET status=TRANSLATED

7. UPDATE job SET status=DONE, completedAt=now()

// Total T+0s to T+18s (typical for a 20-bubble page)

---

*Comic Trans Studio — API Reference / คู่มือ API* | v1.0 | All endpoints prefix: /api