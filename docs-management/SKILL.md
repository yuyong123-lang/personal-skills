---
name: docs-management
description: Use when creating, updating, or organizing documentation in the docs directory. Ensures compliance with project documentation standards including naming conventions, structure, and format requirements.
---

# Documentation Management

## Overview

Manages documentation in the `docs/` directory according to project standards. Enforces naming conventions (yyyy-MM-dd-name.md), directory structure, and content format requirements defined in project documentation standards.

## When to Use

**Use this skill when:**
- Creating new documentation files
- Updating existing documentation
- Organizing documentation structure
- Generating API interface documentation after testing
- Ensuring documentation compliance with project standards

**Don't use for:**
- Code comments or inline documentation
- README files in subdirectories (unless specifically requested)
- Temporary notes or drafts (use scratch files instead)

## Quick Reference

### Document Types and Locations

| Type | Location | Naming Pattern |
|------|----------|----------------|
| API Interface | `docs/interface/` | `yyyy-MM-dd-api-name.md` |
| Architecture | `docs/architecture/` | `yyyy-MM-dd-arch-topic.md` |
| User Guides | `docs/guides/` | `yyyy-MM-dd-guide-name.md` |
| Module Docs | `docs/modules/` | `yyyy-MM-dd-module-name.md` |
| Test Reports | `docs/testing/` | `yyyy-MM-dd-test-report.md` |
| Process Docs | `docs/process/` | `yyyy-MM-dd-process-name.md` |
| SQL Scripts | `docs/sql/` | `yyyy-MM-dd-script-purpose.sql` |
| Summaries | `docs/summaries/` | `yyyy-MM-dd-summary-topic.md` |
| Optimization | `docs/optimization/` | `yyyy-MM-dd-optimization-topic.md` |

### Naming Convention

**Format:** `yyyy-MM-dd-document-name.md`

**Rules:**
- Date prefix: Creation date in `yyyy-MM-dd` format
- Name: English only, lowercase, hyphen-separated
- Extension: `.md` for markdown files

**Examples:**
```
✅ 2026-02-10-user-authentication-api.md
✅ 2026-02-10-database-optimization-plan.md
❌ api-documentation.md (missing date)
❌ 2026-2-10-api.md (date format incorrect)
❌ 2026-02-10-API文档.md (contains Chinese)
```

## Document Structure Template

### Standard Document Header

```markdown
# Document Title

**Author**: Dylan
**创建时间**: yyyy-MM-dd
**最后更新**: yyyy-MM-dd
**版本**: v1.0
**状态**: [草稿|审核中|已发布|已归档]

---

## 目录

[Table of contents]

---

## 概述

[Brief overview]

---

## [Main Content Sections]

---

## 附录

[Optional: glossary, references, change history]

---

**文档维护人**: Dylan
**最后更新**: yyyy-MM-dd
**版本**: v1.0
```

### API Interface Documentation Template

```markdown
# API Name

**Author**: Dylan
**创建时间**: yyyy-MM-dd
**最后更新**: yyyy-MM-dd

---

## 接口名称

### 基本信息

- **接口路径**: `/api/v1/resource`
- **请求方法**: `GET`
- **接口描述**: Description
- **认证方式**: JWT Token

### 请求参数

#### 路径参数

| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| id    | Long | 是   | Resource ID |

#### 查询参数

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|-------|------|------|--------|------|
| page  | Integer | 否 | 1 | 页码 |

#### 请求体

```json
{
  "field": "value"
}
```

### 响应结果

#### 成功响应

**HTTP 状态码**: 200

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {}
}
```

#### 错误响应

**HTTP 状态码**: 400

```json
{
  "code": "ERROR_CODE",
  "message": "错误信息",
  "data": null
}
```

### 调用示例

```bash
curl -X GET "http://localhost:8080/api/v1/resource" \
  -H "Authorization: Bearer {token}"
```

### 注意事项

- Important notes
- Performance considerations
- Security notes
```

## Implementation Workflow

### Creating New Documentation

1. **Determine document type and location**
   - Check Quick Reference table for correct directory
   - Verify directory exists: `ls docs/[category]/`

2. **Generate filename**
   - Use current date: `yyyy-MM-dd`
   - Create descriptive English name (lowercase, hyphen-separated)
   - Example: `2026-02-10-task-management-api.md`

3. **Create document with template**
   - Use appropriate template (standard or API)
   - Fill in header metadata
   - Add author signature: `@author Dylan`

4. **Write content**
   - Follow markdown formatting standards
   - Use proper heading hierarchy
   - Add code blocks with language specification
   - Include examples where appropriate

5. **Verify compliance**
   - Filename matches pattern
   - Document in correct directory
   - Header metadata complete
   - Content follows structure template

### Updating Existing Documentation

1. **Read current document**
   - Use Read tool to view existing content
   - Note current version number

2. **Make updates**
   - Use Edit tool for modifications
   - Update "最后更新" date
   - Increment version if significant changes

3. **Update change history** (if present)
   - Add entry to change history table
   - Document what changed and why

### Generating API Documentation After Testing

**IMPORTANT**: After completing controller tests, generate corresponding API documentation.

1. **Review test code**
   - Identify all tested endpoints
   - Extract request/response examples from tests
   - Note authentication requirements

2. **Create API document**
   - Location: `docs/interface/`
   - Filename: `yyyy-MM-dd-[controller-name]-api.md`
   - Use API template

3. **Document each endpoint**
   - Path, method, description
   - Request parameters (path, query, body)
   - Response formats (success and error)
   - Real curl examples

4. **Add notes section**
   - Special considerations
   - Performance notes
   - Security requirements

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Missing date prefix | Add `yyyy-MM-dd-` to filename |
| Chinese in filename | Use English equivalent with hyphens |
| Wrong directory | Move to correct location per Quick Reference |
| Missing author | Add `**Author**: Dylan` to header |
| No version info | Add version and date metadata |
| Incomplete API docs | Use full template with all sections |
| Missing code language | Add language to code blocks: ```java |

## Verification Checklist

Before committing documentation:

- [ ] Filename follows `yyyy-MM-dd-name.md` pattern
- [ ] File in correct `docs/` subdirectory
- [ ] Header includes author, dates, version
- [ ] Content follows appropriate template
- [ ] Code blocks specify language
- [ ] Tables formatted correctly
- [ ] Links are valid (relative paths for internal)
- [ ] No spelling errors in English sections
- [ ] Change history updated (if applicable)

## Real-World Impact

**Consistency**: All documentation follows same naming and structure, making it easy to find and understand.

**Discoverability**: Date-prefixed filenames enable chronological sorting and quick identification of recent documentation.

**Completeness**: Templates ensure no critical sections are missed, especially for API documentation.

**Maintainability**: Clear structure and metadata make it easy to update and version documentation over time.
