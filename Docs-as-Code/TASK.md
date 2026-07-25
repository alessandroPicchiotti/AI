# TASK — Ottimizzazione cold start (primo avvio)

Checklist riutilizzabile su **tutte le istanze GOW derivate da questo template**.
Copiare questo file nella root dell'istanza e lavorarci sopra: ogni intervento ha
verifica preliminare, edit esatto, rischio e rollback.

I dati numerici citati sono quelli misurati sull'istanza `D:\WRKPRV\GOW\PWA4UI\GOW`
(luglio 2026) e servono da riferimento: **rieseguire sempre le verifiche**, perché
ogni istanza ha un `bin/` e un set di view diversi.

Ordine consigliato: T0 → T1 → T2 → T3 → T4 (sviluppo), poi T5 → T6 → T7 (produzione).

---

## Verifica preliminare — da rieseguire in ogni istanza

```bash
# 1. Quante view vengono compilate a runtime? (costo dominante del primo avvio)
find . -name '*.aspx' -not -path './obj/*' | wc -l
find . -name '*.ascx' -not -path './obj/*' | wc -l
find . -name '*.Master' -not -path './obj/*' | wc -l
find . -name '*.cshtml' -not -path './obj/*' | wc -l

# 2. Quanto pesa bin/ (shadow copy ad ogni avvio dell'AppDomain)
cd NTSPJ.EShop.WebUI_CUBE/bin && ls *.dll | wc -l && du -sh .
du -ch DevExpress*.dll 2>/dev/null | tail -1

# 3. DLL pesanti realmente referenziate?
grep -o '<Reference Include="[^"]*"' NTSPJ.EShop.WebUI_CUBE/NTSPJ.EShop.WebUI_CUBE.csproj
grep -rn "DevExpress" --include=*.cs --include=*.cshtml --include=*.aspx --include=*.config . | grep -v "/obj/"

# 4. Stato attuale della configurazione di compilazione
grep -n "compilation debug" NTSPJ.EShop.WebUI_CUBE/Web.config
grep -n "MvcBuildViews" NTSPJ.EShop.WebUI_CUBE/NTSPJ.EShop.WebUI_CUBE.csproj
grep -n "idleTimeout\|applicationPoolDefaults" .vs/*/config/applicationhost.config
```

Riferimento istanza `PWA4UI\GOW`: 122 `.aspx` + 10 `.ascx` + 3 `.Master` + 54 `.cshtml`
= **189 view compilate a runtime**; `bin/` = **163 DLL / 324 MB**; `MvcBuildViews=false`;
`debug="true"`; nessun `optimizeCompilations`; nessun `idleTimeout`.

---

## SVILUPPO LOCALE

### T0 — Rimuovere da `bin/` le DLL non referenziate  ☐

**Problema.** Sull'istanza di riferimento `bin/` conteneva **45 DLL DevExpress v23.2 per
162 MB** (`BonusSkins`, `XtraEditors`, `XtraCharts.Wizard`, `Images`, `RichEdit.Core`,
`Spreadsheet.Core` — componenti **WinForms desktop**) senza **un solo riferimento** in
tutta la solution: nessun `<Reference Include="DevExpress...">` nei tre `.csproj`,
nessuna occorrenza in `.cs`/`.cshtml`/`.aspx`/`.config`.

Conta perché ASP.NET esegue lo **shadow copy dell'intera cartella `bin`** in
`Temporary ASP.NET Files` ad ogni avvio dell'AppDomain — quindi **ad ogni rebuild**,
non solo al primo F5. In più il probing dell'assembly resolver scandisce l'intero set.

**Intervento.** Spostare (non cancellare) le DLL orfane:

```bash
mkdir -p NTSPJ.EShop.WebUI_CUBE/bin_unused
mv NTSPJ.EShop.WebUI_CUBE/bin/DevExpress*.dll NTSPJ.EShop.WebUI_CUBE/bin_unused/
```

**Verifica.** Rebuild → l'app parte → testare in particolare **stampe e export Telerik**
(`Telerik.Reporting`, `Telerik.ReportViewer.WebForms`), unico punto plausibile di
caricamento dinamico.

**Rischio.** Basso ma non nullo: un caricamento via reflection non intercettato dal grep.
**Rollback.** `mv bin_unused/*.dll bin/`.

**Nota collaterale.** Quando l'istanza verrà messa sotto git, `bin/` va escluso dal
versionamento a prescindere.

---

### T1 — `optimizeCompilations="true"`  ☐

**Problema.** Senza questo flag, ogni rebuild delle DLL invalida l'**intera** cache di
codegen e ASP.NET ricompila tutte le view (189 sull'istanza di riferimento).

**Intervento.** `NTSPJ.EShop.WebUI_CUBE/Web.config`:

```xml
<compilation debug="true" targetFramework="4.8" optimizeCompilations="true">
```

Ricompila solo le view effettivamente modificate. È il singolo intervento più efficace
sul ciclo build → verifica in browser.

**Controindicazione nota.** Se cambia la classe base di un `.Master`/layout, o firme usate
inline dalle view, il codegen può restare stantìo → sintomi strani e incoerenti.
**Rimedio:** svuotare `%LOCALAPPDATA%\Temp\Temporary ASP.NET Files`.

---

### T2 — `batch="false"` (SOLO in locale)  ☐

**Problema.** Con `batch="true"` (default) il primo hit su una cartella compila **tutte**
le pagine di quella cartella, non solo quella richiesta.

**Intervento.** `Web.config` locale:

```xml
<compilation debug="true" batch="false" optimizeCompilations="true">
```

Adatto al flusso "build → apro una pagina → verifico". **Non deployare in produzione**:
genera una assembly per pagina e degrada il throughput.

---

### T3 — App pool IIS Express che non muore  ☐

**Problema.** Senza `idleTimeout` esplicito, l'app pool si spegne dopo ~20 min di
inattività e il warm-up si ripaga da capo.

**Intervento.** `.vs\<nome-solution>\config\applicationhost.config`, dentro
`<applicationPools>`:

```xml
<applicationPoolDefaults managedRuntimeVersion="v4.0">
    <processModel loadUserProfile="true" setProfileEnvironment="false"
                  idleTimeout="00:00:00" idleTimeoutAction="Suspend" />
    <recycling><periodicRestart time="00:00:00" /></recycling>
</applicationPoolDefaults>
```

**Attenzione.** Individuare l'`applicationhost.config` **realmente in uso** dall'istanza
(su `PWA4UI\GOW` è `.vs\Gow-4\config\applicationhost.config`, porta 61270; esiste anche
un file legacy su porta 10991 che punta altrove → ignorarlo). Il file è dentro `.vs\`,
quindi l'edit va rifatto se la cartella viene rigenerata.

---

### T4 — Warm-up automatico post-build  ☐

**Intervento.** Hook `PostToolUse` o target MSBuild che dopo la build fa una richiesta a
`http://localhost:<porta>/`, così la compilazione runtime avviene in background invece
che quando apri il browser. Si integra con la skill `gow-verify-page`.

---

## PRODUZIONE

### T5 — Precompilazione in publish  ☐

**Intervento risolutivo per il cold start in produzione.** `aspnet_compiler -u-` oppure
"Precompila durante la pubblicazione" + *non aggiornabile* nel publish profile.
Elimina completamente la compilazione runtime delle view: il primo hit resta solo JIT.

Beneficio collaterale: gli errori Razor/ASPX emergono a build time invece che aprendo
la pagina. (Alternativa parziale già presente nel `.csproj`: `MvcBuildViews=true`, che
però rallenta ogni build locale.)

---

### T6 — `debug="false"` nel `Web.config` deployato  ☐

Il `Web.config` versionato ha `debug="true"`. In produzione significa niente batch
optimization, niente caching aggressivo del codegen e cold start penalizzato ad ogni
riciclo. Da forzare via `Web.Release.config` / transform, non a mano.

---

### T7 — IIS Application Initialization  ☐

Sul server IIS: `startMode="AlwaysRunning"` sull'app pool + `preloadEnabled="true"`
sull'applicazione + `idleTimeout="0"` + riciclo periodico disabilitato.
Il primo avvio dopo deploy/riciclo lo paga IIS, non il primo utente.

---

## FUORI TEMA MA CORRELATO AL PRIMO HIT

### T8 — `Application_Error` chiama `EventLog.SourceExists` ad ogni errore  ☐

`Global.asax.cs` (≈ riga 63): `EventLog.SourceExists(source)` viene invocato **ad ogni**
errore applicativo, non solo al primo. Enumera il registro (lento) e, se il source non
esiste e il worker process non è admin, `CreateEventSource` lancia `SecurityException`
**dentro** l'error handler, mascherando l'eccezione originale.

**Intervento.** Creare l'event source una volta in fase di installazione; nel codice
lasciare solo `WriteEntry` dentro try/catch.

---

## Misurazione — prima/dopo

Protocollo minimo per avere numeri invece che impressioni:

1. Fermare IIS Express, `Remove-Item -Recurse -Force "$env:LOCALAPPDATA\Temp\Temporary ASP.NET Files\*"`
2. Rebuild completo
3. `Measure-Command { Invoke-WebRequest http://localhost:<porta>/ -UseBasicParsing }`
4. Ripetere 3 volte, tenere la mediana

Registrare i risultati qui sotto, per istanza.

| Istanza | Data | Cold start prima | Cold start dopo | Interventi applicati |
|---------|------|------------------|-----------------|----------------------|
| `PWA4UI\GOW` | — | da misurare | da misurare | nessuno |
