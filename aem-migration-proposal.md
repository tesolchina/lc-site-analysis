# AEM Migration Proposal: Automated Content Migration via Adobe Experience Manager API

_Proposal for migrating LC WordPress site to Adobe Experience Manager (AEM)_
_Date: 2026-08-27_
_Status: Draft for review_

---

## 1. Executive Summary

This proposal outlines an automated migration strategy from the current WordPress-based LC website to Adobe Experience Manager (AEM). By leveraging the AEM REST API and our existing WordPress API access, we can programmatically migrate 300+ pages, reducing manual effort by an estimated 70-80%.

**Key benefits:**
- Automated bulk migration of pages, media, and structure
- Preservation of SEO (URLs, metadata, redirects)
- Minimal downtime during cutover
- Repeatable process for future content updates

---

## 2. Current State Analysis

### WordPress Site Inventory (from API analysis)

| Metric | Count | Notes |
|---|---|---|
| **Total pages** | 300 | All published |
| **Top-level pages** | 189 | No parent |
| **Pages with children** | 16 | Hierarchical structure |
| **Flagged pages** | 48 | PDF/image only, need content |
| **Empty pages** | 5 | <100 chars content |
| **Media files** | ~500+ | Images, PDFs, videos |

### Content Types Identified

| Type | Count | Migration Complexity |
|---|---|---|
| Standard pages | ~200 | Low — direct content transfer |
| Staff profiles | ~50 | Medium — structured data |
| Event/seminar pages | ~30 | Medium — dates, locations |
| Competition pages | ~20 | Medium — winners, rules |
| PDF/image-only pages | 48 | **High** — need content creation |
| Photo galleries | ~30 | Medium — media migration |

---

## 3. AEM API Capabilities

### Core AEM APIs for Migration

| API | Endpoint | Purpose |
|---|---|---|
| **Content Fragments** | `/api/assets/contentfragments` | Structured content (staff, events) |
| **Pages** | `/api/pages` | Page creation and management |
| **Assets** | `/api/assets` | Media upload and organization |
| **Templates** | `/api/templates` | Page templates |
| **Workflow** | `/api/workflow` | Approval processes |

### Key AEM Features for Migration

1. **Content Fragment Models** — Define structured content types (staff, events, courses)
2. **Editable Templates** — Consistent page layouts
3. **Asset Management** — Organized media library with metadata
4. **Multi-site Manager (MSM)** — Language variants (EN/中文)
5. **Workflow** — Content approval before publish

---

## 4. Migration Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  WordPress API  │────▶│  Migration Script │────▶│   AEM API       │
│  (Source)       │     │  (Python/Node)    │     │   (Target)      │
└─────────────────┘     └──────────────────┘     └─────────────────┘
        │                        │                        │
        ▼                        ▼                        ▼
   • Pages (300)            • Transform              • Pages
   • Media (500+)           • Map fields             • Assets
   • Structure              • Validate               • Content Fragments
   • Metadata               • Error handling         • Workflows
```

---

## 5. Migration Phases

### Phase 1: Preparation (Week 1-2)

| Task | Owner | Deliverable |
|---|---|---|
| Set up AEM development environment | AEM admin | Dev instance |
| Define Content Fragment Models | Content architect | CF models for staff, events, courses |
| Create page templates | AEM developer | Templates matching current layouts |
| Map WordPress → AEM fields | Migration lead | Field mapping document |
| Test AEM API access | Developer | API authentication working |

### Phase 2: Media Migration (Week 2-3)

| Task | Tool | Notes |
|---|---|---|
| Export media from WordPress | WP API | ~500 files |
| Upload to AEM Assets | AEM Assets API | Organized by type/year |
| Update media references | Migration script | Fix URLs in content |
| Verify media integrity | Automated check | File size, format |

### Phase 3: Content Migration (Week 3-5)

| Task | Tool | Notes |
|---|---|---|
| Migrate standard pages | Migration script | ~200 pages |
| Migrate staff profiles | Migration script | ~50 pages → Content Fragments |
| Migrate event pages | Migration script | ~30 pages → Content Fragments |
| Create missing content | Manual + AI assist | 48 flagged pages |
| Set up redirects | AEM Redirect Manager | Old URLs → new URLs |

### Phase 4: Testing & Cutover (Week 5-6)

| Task | Owner | Notes |
|---|---|---|
| Content review | LC staff | Sample pages |
| Link checking | Automated | Internal + external |
| SEO verification | SEO tool | Meta tags, structured data |
| Performance testing | Load testing | Page speed |
| Cutover | AEM admin | DNS switch |
| Post-cutover monitoring | DevOps | 1 week |

---

## 6. Technical Implementation

### 6.1 Migration Script Structure

```python
# migration_script.py
import requests
import json
from typing import Dict, List

class AEMMigrator:
    def __init__(self, wp_api_url: str, aem_api_url: str, aem_auth: tuple):
        self.wp_api = wp_api_url
        self.aem_api = aem_api_url
        self.aem_auth = aem_auth
        
    def migrate_page(self, wp_page_id: int) -> Dict:
        """Migrate a single WordPress page to AEM"""
        # 1. Fetch from WordPress
        wp_page = self.fetch_wp_page(wp_page_id)
        
        # 2. Transform content
        aem_content = self.transform_content(wp_page)
        
        # 3. Create in AEM
        aem_page = self.create_aem_page(aem_content)
        
        # 4. Migrate media
        self.migrate_media(wp_page, aem_page)
        
        # 5. Set up redirect
        self.create_redirect(wp_page['link'], aem_page['path'])
        
        return aem_page
    
    def transform_content(self, wp_page: Dict) -> Dict:
        """Transform WordPress content to AEM format"""
        # Map WordPress fields to AEM fields
        # Handle Elementor → AEM component conversion
        # Preserve SEO metadata
        pass
    
    def create_aem_page(self, content: Dict) -> Dict:
        """Create page in AEM via API"""
        # POST to AEM Pages API
        # Handle templates, components
        pass
```

### 6.2 Field Mapping (WordPress → AEM)

| WordPress Field | AEM Field | Notes |
|---|---|---|
| `title.rendered` | `jcr:title` | Page title |
| `content.rendered` | `jcr:content` | Main content |
| `slug` | `pageName` | URL-friendly name |
| `link` | `cq:redirectTarget` | For redirects |
| `date` | `jcr:created` | Creation date |
| `modified` | `jcr:lastModified` | Last modified |
| `parent` | `parentPage` | Hierarchy |
| `meta._elementor_data` | `components` | Elementor → AEM components |
| `featured_media` | `thumbnail` | Featured image |

### 6.3 Component Mapping (Elementor → AEM)

| Elementor Widget | AEM Component | Notes |
|---|---|---|
| `heading` | `title` | Text heading |
| `text-editor` | `text` | Rich text |
| `image` | `image` | Single image |
| `image-carousel` | `carousel` | Image slider |
| `button` | `button` | CTA button |
| `video` | `video` | Video player |
| `icon-box` | `teaser` | Icon + text |
| `html` | `html` | Raw HTML |
| `section` | `container` | Layout container |
| `column` | `column` | Layout column |

---

## 7. Handling Flagged Pages (48 pages)

### Strategy for PDF/Image-Only Pages

| Approach | Count | Effort | Notes |
|---|---|---|---|
| **Create proper pages** | ~30 | High | Write actual content |
| **Keep as media galleries** | ~10 | Medium | AEM Media Gallery component |
| **Archive/remove** | ~8 | Low | Outdated events |

### Content Creation Workflow

1. **AI-assisted drafting** — Use AI to generate initial content from PDFs/images
2. **Staff review** — LC staff review and edit
3. **AEM workflow** — Approval before publish
4. **SEO optimization** — Meta tags, descriptions

---

## 8. SEO Preservation

### URL Structure

| Current (WordPress) | New (AEM) | Action |
|---|---|---|
| `/main/ielts/` | `/content/lc/en/ielts` | 301 redirect |
| `/main/staff-english/` | `/content/lc/en/people/english` | 301 redirect |
| `/main/symposium_25-june-2026/` | `/content/lc/en/events/symposium-2026` | 301 redirect |

### Metadata Migration

| WordPress | AEM | Notes |
|---|---|---|
| Yoast SEO title | `pageTitle` | Page title |
| Yoast meta description | `jcr:description` | Meta description |
| Open Graph tags | `og:*` properties | Social sharing |
| Structured data | `schema.org` markup | Rich snippets |

---

## 9. Risk Mitigation

| Risk | Mitigation |
|---|---|
| **Data loss** | Full backup before migration; incremental sync |
| **Downtime** | Staged migration; DNS cutover during low traffic |
| **Broken links** | Automated link checking; redirect mapping |
| **Content formatting** | Component mapping; manual review of complex pages |
| **SEO impact** | 301 redirects; preserve metadata; monitor rankings |
| **Staff training** | AEM training sessions; documentation |

---

## 10. Timeline & Resources

### Estimated Timeline: 6 weeks

| Week | Phase | Key Deliverables |
|---|---|---|
| 1-2 | Preparation | AEM setup, CF models, templates |
| 2-3 | Media Migration | All media in AEM Assets |
| 3-5 | Content Migration | All pages migrated |
| 5-6 | Testing & Cutover | QA, redirects, go-live |

### Resource Requirements

| Role | Allocation | Responsibilities |
|---|---|---|
| AEM Developer | 50% | API integration, component mapping |
| Content Architect | 25% | CF models, templates |
| Migration Lead | 25% | Script development, testing |
| LC Staff | 20% | Content review, approval |
| SEO Specialist | 10% | Redirects, metadata |

---

## 11. Cost Estimate

| Item | Cost (HKD) | Notes |
|---|---|---|
| AEM licensing | Existing | HKBU institutional |
| Development | ~80,000 | 4 weeks @ 20k/week |
| Content creation | ~30,000 | 48 pages @ ~600/page |
| Testing/QA | ~20,000 | 1 week |
| Contingency | ~20,000 | 15% buffer |
| **Total** | **~150,000** | |

---

## 12. Success Metrics

| Metric | Target | Measurement |
|---|---|---|
| Pages migrated | 100% | Count |
| Media files migrated | 100% | Count |
| Broken links | <1% | Link checker |
| Page load time | <3s | Performance test |
| SEO ranking preservation | >90% | Rank tracking |
| Staff satisfaction | >80% | Survey |

---

## 13. Next Steps

1. **Approve proposal** — LC management + IT
2. **Set up AEM dev environment** — IT + AEM admin
3. **Define Content Fragment Models** — Content architect
4. **Develop migration script** — Developer
5. **Pilot migration** — 10 test pages
6. **Full migration** — All 300 pages
7. **Cutover** — DNS switch
8. **Post-migration support** — 1 month

---

## Appendix A: API Endpoints

### WordPress API (Source)
```
GET /wp-json/wp/v2/pages?per_page=100
GET /wp-json/wp/v2/pages/{id}
GET /wp-json/wp/v2/media
GET /wp-json/lc/v1/page/{id}  (custom plugin)
```

### AEM API (Target)
```
POST /api/pages
POST /api/assets
POST /api/contentfragments
GET /api/templates
POST /api/workflow
```

---

## Appendix B: Sample Migration Script

See `projects/LCwebsite/migration-script.py` for full implementation.

---

_Proposal prepared by: AI Agent_
_Reviewed by: [Pending]_
_Approved by: [Pending]_
