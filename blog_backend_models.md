# هسته‌ی محتوا (Core)

## 1) User

**هدف:** حساب کاربری همه‌ی افراد (ادمین، نویسنده، ادیتور، خواننده‌ی لاگین‌کرده).  
**فیلدها:**  
- id (PK)، email (unique, indexed)، password_hash، name  
- status (active/suspended)  
- created_at, updated_at, last_login_at, last_ip  

**روابط:** 1–N با Post, Comment; N–N با Role  
**ایندکس‌ها:** (email)، (status)  
**نکات:** MFA/2FA، rate limit برای لاگین، password reset tokens.

---

## 2) Role و 3) Permission (یا RolePermission)

**هدف:** کنترل دسترسی.  
**فیلدها:**  
- Role: id, name (admin/editor/author/reader), description  
- Permission: id, code (e.g. post.create, post.publish)  
- RolePermission: role_id, permission_id (PK مرکب)

**روابط:** UserRole (N–N) بین User و Role.  
**ایندکس‌ها:** (name)، (code)  
**نکات:** حداقل نقش‌ها: Admin, Editor, Author, Viewer.

---

## 4) AuthorProfile

**هدف:** پروفایل نویسنده جدا از User (bio، لینک‌ها).  
**فیلدها:** user_id (unique), display_name, bio, avatar_id (Media)، social_links (jsonb)  
**روابط:** 1–1 با User، N–1 از Post به AuthorProfile.  
**ایندکس‌ها:** (display_name)

---

## 5) Category

**هدف:** دسته‌بندی سلسله‌مراتبی.  
**فیلدها:** id, slug (unique), name, parent_id (nullable), description, order  
**روابط:** Self-relation (parent/children)، N–1 از Post به Category (یا N–N اگر چنددسته‌ای).  
**ایندکس‌ها:** (slug), (parent_id, order)

---

## 6) Tag

**هدف:** برچسب‌های آزاد.  
**فیلدها:** id, slug (unique), name, description  
**روابط:** N–N با Post از طریق PostTag.  
**ایندکس‌ها:** (slug), (name)

---

## 7) Post

**هدف:** مقاله/پست وبلاگ.  
**فیلدها:**  
- هویت/URL: id, slug (unique), canonical_url (nullable)  
- متن: title, excerpt, content (rich)، reading_time_sec  
- وضعیت: status (draft/review/scheduled/published/archived), visibility (public/private/unlisted)  
- زمان‌ها: published_at (indexed), scheduled_at  
- ارجاعات: author_id, category_id (nullable), series_id (nullable), cover_media_id (nullable)  
- SEO: seo_title, seo_description, og_image_id (nullable)  
- آمار: views_count, likes_count, comments_count (اختیاری و قابل‌مشتق)

**روابط:** N–1 به AuthorProfile; N–N به Tag; N–1 به Category; N–1 به Series  
**ایندکس‌ها:** (slug), (status, published_at), (category_id, published_at)  
**نکات:** نگهداری content_html کش‌شده برای رندر سریع (از Markdown → HTML).

---

## 8) PostTag (جدول واسط)

**فیلدها:** post_id + tag_id (PK مرکب), created_at  
**ایندکس‌ها:** (tag_id, post_id)

---

## 9) Series

**هدف:** مجموعه مقالات پیوسته.  
**فیلدها:** id, slug (unique), title, description, order_strategy (manual|by_date)  
**روابط:** 1–N با Post  
**ایندکس‌ها:** (slug)

---

## 10) Media

**هدف:** مدیریت فایل‌ها (عکس، ویدیو، صوت).  
**فیلدها:** id, storage_key, url, type (image/video/audio/file), mime, width, height, duration, size_bytes, alt_text, title, uploaded_by, created_at  
**روابط:** N–1 به User (آپلودکننده)، ارجاع از Post, AuthorProfile, SEO/OpenGraph  
**ایندکس‌ها:** (type), (uploaded_by, created_at)

---

# انتشار، ویرایش، تعامل

## 11) Revision

**هدف:** تاریخچه نسخه‌های پست/صفحات (audit content).  
**فیلدها:** id, post_id, editor_id, content, title, excerpt, change_note, created_at  
**ایندکس‌ها:** (post_id, created_at DESC)  
**نکات:** امکان rollback.

---

## 12) Comment

**هدف:** دیدگاه‌ها.  
**فیلدها:**  
- id, post_id (indexed), user_id (nullable اگر مهمان), parent_id (nullable برای threading)  
- author_name/author_email (برای مهمان‌ها)، content, status (pending/approved/spam/removed)  
- created_at, ip, user_agent  

**ایندکس‌ها:** (post_id, status, created_at)، (parent_id)  
**نکات:** فیلتر اسپم، نرخ ارسال، قابلیت بستن نظرات هر پست.

---

## 13) Reaction

**هدف:** لایک/ایموجی روی پست یا کامنت.  
**فیلدها:** id, target_type (post|comment), target_id, user_id (nullable), reaction (like|emoji_code), created_at, ip  
**ایندکس‌ها:** (target_type, target_id, reaction)، یکتا روی (target_type, target_id, user_id, reaction)  
**نکات:** برای کاربران ناشناس می‌شود با کوکی/فینگرپرینت محدود کرد.

---

## 14) Page

**هدف:** صفحات استاتیک مثل «درباره ما/تماس».  
**فیلدها:** مشابه Post ولی ساده‌تر: slug (unique), title, content, status, published_at, seo_*  
**ایندکس‌ها:** (slug), (status, published_at)

---

## 15) Menu / MenuItem

**هدف:** منوهای ناوبری قابل‌مدیریت.  
**فیلدها:**  
- Menu: id, name, location (header|footer|sidebar)  
- MenuItem: id, menu_id, parent_id (nullable), label, url, order, target_blank  

**ایندکس‌ها:** (menu_id, parent_id, order)
