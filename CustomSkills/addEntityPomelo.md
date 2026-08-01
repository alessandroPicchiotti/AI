# .claude/skills/crud-layer/SKILL.md
---
name: crud-layer
description: Collega un nuovo campo o entità attraverso tutti i layer MVC di questa soluzione .NET legacy
---
Quando dato un nome entity/field:
1. Individua la tabella DB e genera lo script T-SQL ALTER/CREATE.
2. Aggiorna in ordine: Entity → Repository → ViewModel → Controller → view Razor/ASPX.
3. Usa email/number input types + validazione client-side dove rilevante.
4. Scrivi tutti i file come UTF-8 CON BOM.
5. Usa jQuery(...) esplicitamente, mai bare $.
6. Esegui la build della soluzione e incolla il risultato.
7. Elenca ogni file toccato al fine.
