# Lab05 — Хичээлд бүртгүүлэх API-г Postman/Newman-аар тестлэх

| | |
|---|---|
| **Нэр** | `<Х.Дэлгэр>` |
| **Оюутны код** | `<B222270835>` |
| **node -v** | `<v24.13.0>` |
| **newman -v** | `<6.2.2>` |

## Ажиллуулах

```powershell
node server.js                     # 1-р терминал: http://localhost:3000
newman run lab05-collection.json      --env-var baseUrl=http://localhost:3000
newman run lab05-collection-fail.json --env-var baseUrl=http://localhost:3000
```

## Репозиторийн бүтэц

| Файл | Агуулга |
|---|---|
| `server.js` | Тестлэгдэх бүртгэлийн API (`POST /registrations`) болон setup endpoint-ууд (`PUT /students/:id`, `PUT /courses/:id`) |
| `lab05-collection.json` | 10 бие даасан тест, 24 assertion — бүгд PASS байх ёстой |
| `lab05-collection-fail.json` | Ижил collection, гэхдээ 1-р тестийн статусын oracle-ийг зориуд буруу (200) болгосон |
| `results/newman-pass.txt` | Үндсэн collection-ы ажиллагаа (сервер асаалттай) |
| `results/newman-fail.txt` | Fail collection-ы ажиллагаа (сервер асаалттай) |
| `results/newman-down.txt` | Үндсэн collection-ы ажиллагаа (сервер унтраалттай) |

## Сонголт / утгын хүснэгт

`POST /registrations`-ийн оролтын нөхцөл бүрийн сонголт ба түүнд харгалзах хариу:

| Хүчин зүйл | Сонголт | Хүлээгдэх хариу |
|---|---|---|
| Хүсэлтийн бие | `studentID` ба `courseID` хоёулаа байгаа | Цааш шалгана |
| | `courseID` (эсвэл `studentID`) дутуу | `400`, `ERROR_BAD_REQUEST` |
| Оюутан | Бүртгэлтэй | Цааш шалгана |
| | Бүртгэлгүй | `200`, `ERROR_NO_STUDENT` |
| Оюутны төлөв | `active` | Цааш шалгана |
| | `inactive` | `200`, `ERROR_INACTIVE_STUDENT` |
| Хичээл | Бүртгэлтэй | Цааш шалгана |
| | Бүртгэлгүй | `200`, `ERROR_NO_COURSE` |
| Өмнөх холбоо (prerequisite) | Бүгдийг нь өгсөн, эсвэл prerequisite байхгүй | `201`, `OK`, `registrationID` (тоо) |
| | Нэгийг ч өгөөгүй | `200`, `ERROR_PREREQUISITES`, `missing` = бүх prerequisite |
| | Заримыг нь өгсөн | `200`, `ERROR_PREREQUISITES`, `missing` = дутуу хэсэг |

Шалгах дараалал: бие → оюутан → төлөв → хичээл → prerequisite. Хоёр алдаа зэрэг байвал эхний алдаа л буцна (тест 7, 8).

## Спецификацийн хүснэгт (10 тест, 24 assertion)

Тест бүр өөрийн setup-аа (`PUT /students`, `PUT /courses`) хийдэг тул бусад тестээс хамааралгүй, дангаар нь ажиллуулж болно. `registrationID`-ийн яг утгыг шалгаагүй, зөвхөн тоо эсэхийг шалгасан (утга нь өмнө хийсэн бүртгэлийн тооноос хамаардаг).

| # | Тест | Setup | POST бие | Хүлээгдэх статус | Хүлээгдэх `result` | Assertion | Үр дүн |
|---|---|---|---|---|---|---|---|
| 1 | Happy path: prerequisite-тэй идэвхтэй оюутан бүртгүүлнэ | B231234567 active, CS201 өгсөн; CS313 ← CS201 | B231234567, CS313 | 201 | `OK`, `registrationID` тоо | 3 | PASS |
| 2 | Бүртгэлгүй оюутан | CS313 ← CS201 | B000000000, CS313 | 200 | `ERROR_NO_STUDENT` | 2 | PASS |
| 3 | Идэвхгүй оюутан | B231234567 inactive; CS313 ← CS201 | B231234567, CS313 | 200 | `ERROR_INACTIVE_STUDENT` | 2 | PASS |
| 4 | Бүртгэлгүй хичээл | B231234567 active | B231234567, CS999 | 200 | `ERROR_NO_COURSE` | 2 | PASS |
| 5 | Prerequisite огт өгөөгүй | B231234567 active, `[]`; CS313 ← CS201 | B231234567, CS313 | 200 | `ERROR_PREREQUISITES`, `missing = ["CS201"]` | 3 | PASS |
| 6 | Prerequisite-ийн заримыг өгсөн | B231234567 active, CS201; CS313 ← CS201, CS202 | B231234567, CS313 | 200 | `ERROR_PREREQUISITES`, `missing = ["CS202"]` | 3 | PASS |
| 7 | Давхар алдаа: оюутан ч, хичээл ч байхгүй | — | B000000000, CS999 | 200 | `ERROR_NO_STUDENT` (эхнийх) | 2 | PASS |
| 8 | Давхар алдаа: идэвхгүй + prerequisite дутуу | B231234567 inactive, `[]`; CS313 ← CS201 | B231234567, CS313 | 200 | `ERROR_INACTIVE_STUDENT` (эхнийх) | 2 | PASS |
| 9 | Хил: prerequisite-гүй хичээл, хоосон `coursesTaken` | B231234567 active, `[]`; CS313 ← `[]` | B231234567, CS313 | 201 | `OK`, `registrationID` тоо | 3 | PASS |
| 10 | Хил: `courseID` дутуу | B231234567 active; CS313 ← `[]` | B231234567 | 400 | `ERROR_BAD_REQUEST` | 2 | PASS |
| | **Нийт** | | | | | **24** | **24 / 24** |

**Assertion-ы тоо: 24** — `results/newman-pass.txt`-ийн `assertions executed = 24`-тэй таарна.

## Newman-ы үр дүн

| Ажиллагаа | Файл | requests (executed / failed) | assertions (executed / failed) | Exit code |
|---|---|---|---|---|
| Сервер асаалттай, үндсэн collection | `results/newman-pass.txt` | 26 / 0 | 24 / **0** | **0** |
| Сервер асаалттай, fail collection | `results/newman-fail.txt` | 26 / 0 | 24 / **1** | **1** |
| Сервер унтраалттай, үндсэн collection | `results/newman-down.txt` | 26 / **26** | 24 / 24 | **1** |

### Fail collection

`lab05-collection-fail.json`-д 1-р тестийн статусын oracle-ийг `201`-ээс `200` болгож зориуд буруу болгосон. Сервер зөв ажиллаж `201` буцаасан тул Newman `AssertionError: expected response to have status code 200 but got 201` гэж 1 assertion унагаж, exit code 1-ээр дууссан. Энэ нь тест бодит алдааг илрүүлж чадна, мөн CI-д унасан ажиллагаа exit code-оор ялгарна гэдгийг харуулна.

### Сервер унтраалттай — интерфейсийн алдаа ба oracle-ийн алдааны ялгаа

`newman-down.txt`-д хүсэлт бүр `connect ECONNREFUSED 127.0.0.1:3000` гэж унасан (26/26 request failed, exit code 1).

- **Интерфейсийн алдаа** (`newman-down.txt`): сервертэй холбогдож чадаагүй тул хариу огт ирээгүй. Тестийн скриптүүд хоосон хариу дээр ажиллаж `JSONError` / `to have property 'code'` гэж унасан — эдгээр нь API буруу ажилласныг бус, тест ажиллах орчин бэлэн биш байгааг илтгэнэ. Шийдэл нь кодыг засах биш, серверийг асаах.
- **Oracle-ийн алдаа** (`newman-fail.txt`): сервер хариу өгсөн (request failed = 0), харин хүлээгдэж буй утга бодит хариутай таараагүй (`AssertionError`). Энэ нь систем эсвэл тестийн хүлээлт буруу гэсэн үг бөгөөд задлан шинжлэх шаардлагатай.

Хоёулаа exit code 1 өгдөг тул зөвхөн exit code-оор ялгах боломжгүй; `requests failed` мөр болон алдааны төрлийг (`ECONNREFUSED` vs `AssertionError`) харж ялгана.

## Дүгнэлт

Энэ лабораторид хичээлд бүртгүүлэх API-ийн `POST /registrations` функцэд Postman collection бичиж, Newman-аар командын мөрөөс автоматаар ажиллуулсан. Оролтын нөхцөлүүдийг сонголт/утгын хүснэгтээр ангилж, амжилттай тохиолдол, алдааны тохиолдол бүр, давхар алдаа, хилийн утгыг хамарсан 10 тест, 24 assertion гаргасан. Тест бүр өөрийн setup-ыг хийдэг тул дараалал болон бусад тестээс хамааралгүй ажилладаг. Oracle-ууд статус кодыг ч, `result` болон `missing` утгыг ч шалгадаг ба дарааллаас хамаардаг `registrationID`-ийн яг утгыг бус зөвхөн төрлийг нь шалгасан. Үндсэн ажиллагаа 24/24 assertion амжилттай, exit code 0-ээр дууссан. Зориуд буруу oracle-тэй fail collection 1 assertion унагаж exit code 1 өгсөн нь тест бодит зөрүүг илрүүлж чадахыг баталсан. Сервер унтраалттай үед бүх хүсэлт `ECONNREFUSED`-ээр унасан нь интерфейсийн алдаа бөгөөд oracle-ийн алдаанаас `requests failed` тоо болон алдааны төрлөөр ялгагдана. Лекц 5-ын семантикаар оролтын алдаа `200 + ERROR_*` буцдаг тул зөвхөн статус шалгах нь хангалтгүй, `result` утгыг заавал шалгах шаардлагатайг ойлгосон. Newman-ы exit code нь энэ тестийг CI pipeline-д шууд ашиглах боломж олгоно.