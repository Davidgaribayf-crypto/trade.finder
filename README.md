# trade.finder
https://github.com/Davidgaribayf-crypto/trade-finder/new/main
check_links.py
import ssl, sys, urllib.request, urllib.error, datetime

LINKS = [
("IBEW Local 11","https://ibew11.org"),("IBEW Local 6","https://ibew6.org"),("IBEW Local 332","https://ibew332.org"),("IBEW Local 340","https://ibew340.org"),("IBEW Local 569","https://ibew569.org"),("IBEW Local 428","https://ibew428.org"),("IBEW Local 551","https://ibew551.org"),("IBEW Local 684","https://ibew684.org"),("IBEW Local 477","https://ibew477.org"),("IBEW Local 952","https://ibew952.org"),
("UA Local 78","https://ualocal78.org"),("UA Local 38","https://ualocal38.org"),("Carpenters NCCRC","https://nccrc.org"),("Operating Engineers Local 3","https://oe3.org"),
("LA Trade-Tech","https://lattc.edu"),("Cerritos College","https://cerritos.edu"),("CCSF","https://ccsf.edu"),("Fresno City College","https://fresnocitycollege.edu"),("Bakersfield College","https://bakersfieldcollege.edu"),("Riverside City College","https://rcc.edu"),("Shasta College","https://shastacollege.edu"),("College of the Redwoods","https://redwoods.edu"),("Butte College","https://butte.edu"),("Hartnell College","https://hartnell.edu"),("Cuesta College","https://cuesta.edu"),("Santa Barbara City College","https://sbcc.edu"),("College of the Desert","https://collegeofthedesert.edu"),("Imperial Valley College","https://imperial.edu"),("Victor Valley College","https://vvc.edu"),("Lake Tahoe CC","https://ltcc.edu"),("Modesto Junior College","https://yosemite.edu"),("College of the Sequoias","https://cos.edu"),("Oxnard College","https://oxnardcollege.edu"),
]
UA = {"User-Agent": "Mozilla/5.0 (TradeFinder link checker)"}

def fetch(url, method):
    req = urllib.request.Request(url, headers=UA, method=method)
    with urllib.request.urlopen(req, timeout=20, context=ssl.create_default_context()) as r:
        return r.status, r.geturl()

def check(url):
    for method in ("HEAD", "GET"):
        try:
            status, final = fetch(url, method)
            return ("ok" if status < 400 else "broken"), status, final
        except urllib.error.HTTPError as e:
            if method == "GET":
                return "broken", e.code, url
        except Exception as e:
            if method == "GET":
                return "broken", str(e)[:80], url
    return "broken", "unknown", url

def host(u):
    return u.split("//")[-1].split("/")[0].replace("www.", "")

bad, moved, good = [], [], 0
for name, url in LINKS:
    state, code, final = check(url)
    if state == "broken":
        bad.append((name, url, code))
    elif host(final) != host(url):
        moved.append((name, url, final))
    else:
        good += 1

lines = ["# Link check report", "Run date: %s" % datetime.date.today(), "", "Working: %d of %d" % (good, len(LINKS)), ""]
if bad:
    lines += ["## Broken (fix these)"] + ["- %s: %s (%s)" % b for b in bad] + [""]
if moved:
    lines += ["## Redirected to another site (review)"] + ["- %s: %s now goes to %s" % m for m in moved] + [""]
open("report.md", "w").write("\n".join(lines))
print("\n".join(lines))
sys.exit(1 if bad or moved else 0)