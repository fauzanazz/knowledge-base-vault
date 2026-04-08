---
title: "Robots.txt"
category: web-standards
summary: "Robots.txt is a web standard that allows website owners to communicate crawling permissions and restrictions to web crawlers. It serves as a politeness protocol that ethical crawlers must respect."
sources:
  - raw/articles/_done/web-crawler-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:53:13.445Z
---

# Robots.txt

> Robots.txt is a web standard that allows website owners to communicate crawling permissions and restrictions to web crawlers. It serves as a politeness protocol that ethical crawlers must respect.

# Robots.txt

**Robots.txt** is a web standard file that allows website owners to communicate crawling permissions and restrictions to [[Web Crawler]] systems. It serves as a fundamental politeness protocol in web crawling, helping maintain respectful relationships between crawlers and web servers.

## Purpose and Function

Robots.txt files provide instructions to web crawlers about:
- Which parts of a website can be crawled
- Which areas should be avoided
- Crawling delays and frequency limits
- Sitemap locations for efficient discovery

## File Location and Format

The robots.txt file must be located at the root of a website's domain:
```
https://example.com/robots.txt
```

### Basic Syntax
- **User-agent**: Specifies which crawlers the rules apply to
- **Disallow**: Defines paths that should not be crawled
- **Allow**: Explicitly permits crawling of specific paths
- **Crawl-delay**: Sets minimum delay between requests
- **Sitemap**: Points to XML sitemap locations

## Compliance in Web Crawlers

Ethical [[Web Crawler]] systems implement robots.txt compliance by:

1. **Pre-crawl Check**: Fetching and parsing robots.txt before crawling any site
2. **Rule Caching**: Storing robots.txt rules to avoid repeated requests
3. **Path Validation**: Checking each URL against disallow/allow rules
4. **Delay Enforcement**: Respecting crawl-delay directives

## Integration with Crawler Architecture

Robots.txt compliance is typically implemented in the **HTML Downloader** component, which:
- Maintains a cache of robots.txt files per domain
- Validates URLs against stored rules before downloading
- Implements appropriate delays as specified
- Updates cached rules periodically

## Limitations and Considerations

- **Not Legally Binding**: Robots.txt is a voluntary standard
- **Publicly Visible**: Rules are accessible to anyone
- **Cache Management**: Crawlers must balance freshness with efficiency
- **Wildcard Support**: Modern implementations support pattern matching

## Best Practices

For crawler developers:
- Always check robots.txt before crawling
- Implement reasonable default delays even when not specified
- Handle robots.txt fetch failures gracefully
- Respect the spirit of the protocol, not just the letter

Robots.txt compliance is essential for maintaining the health of the web ecosystem and ensuring that [[Web Crawler]] systems operate as good citizens of the internet.

---
*Related: [[Web Crawler]], [[Web Standards]], [[HTTP Protocol]]*
