# DClaw DPanel — Product Specification

> **Status:** In Progress  
> **Owner:** Shell Agent  
> **Priority:** P0

---

## UI Specification

### Footer

The DPanel footer **must** display the current year dynamically:

```
© {current_year} DClawstack. All rights reserved.
```

- **Current year:** 2026
- **Behavior:** Rendered server-side or statically at build time for P0. Dynamic client-side update not required until v1.1.
- **Placement:** Bottom of every DPanel page, below the app grid and billing dashboard.
- **Styling:** `text-sm text-muted-foreground` (Tailwind), centered.

---

*Last updated: 2026-05-04 by Vault Coordinator*
