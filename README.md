<pre>
# MikroTik-dynamic-ipv6

<h2>Magyar verzió</h2>
Ez a könnyen használható szkript segít újraindítani az IPv6-ot a MikroTik eszközödön, kér új prefix-et PPPoE újracsatlakozáskor, és - szükség esetén - képes frissíteni tűzfalszabályokat. Ez hasznos lehet, ha van egy eszköz, például házi szerver, amit az internet felől is elérnénk IPv6-on, de a többi eszközt védenénk a tűzfallal, azaz ott csak az általuk kezdeményezett kapcsolatok mehetnek befelé.

Az IPv6 tűzfallal védett hálózat némileg hasonló az IPv4 NAT-hoz, alapvetően csak a kimenő kapcsolatok és az azokra érkezett válaszok mehetnek befelé. Kivételt képeznek azok az eszközök, amikre külön engedve van a bejövő kapcsolat.

A legtöbb népszerű otthoni router nem támogatja ezt, a dinamikus prefixekhez nem igazán tudnak igazodni. Az új előtag lekérésével általában nincs probléma, de a tűzfalat már nem tudják módosítani. Ez akkor nem probléma, ha minden eszközt egy kalap alá vennénk, azaz vagy beengedünk mindent, vagy limitálunk, de itt most pont egy-egy eszköznek engednénk a bejövő kapcsolatait.

<h3>Beüzemelés</h3>
A beüzemelés néhány rövid lépésből áll. Első körben tegyük vágólapra a <a href="https://github.com/adns44/MikroTik-dynamic-ipv6/blob/main/MikroTik%20dynamic%20IPv6%20script">szkript</a> tartalmát, majd lépjünk be a MikroTik routerbe. Vagy webfelületen, vagy a CLI-n esetleg WinBox-on lépjünk a system > script részre, és adjunk hozzá újat. Engedélyből read és write mindenképp kell neki, a többit még nem sikerült 100% pontossággal belőnöm, így nekem perpill minden engedélyezve van.
Grafikus felületen a szerkesztés egyszerű, CLI-n pedig javaslom az edit parancs használatát, így egy nano-hoz hasonló szerkesztővel lehet a szkript tartalmát kezelni. Tehát először hozzáadás, majd utána edit <id> source (ahol ID az új tétel ID-je).
Másoljuk be a vágólapról. Ezt követően szükség lesz némi módosításra a megfelelő működés érdekében.

Az "#Array with machine names and address suffix pairs" sor alatt találjuk a klienseket tartalmazó tömböt. Ebbe "<kliensnev>"="<suffix>" formában lehet felsorolni az elemeket. Ügyeljünk rá, hogy csak a második 64 bit (/64, és az elején kettőspont nélkül) szerepeljen.
Példa két gépre:
:local machines ({"srv01"="dd44:dd44:dd44:dd44";"srv02"="aa11:bb22:cc33:dd44"})
A példa továbbővíthető, ;-vel tagolva. Az itt lévő gépneveknek szerepelnie kell az IPv6-os tűzfal addresslist-jében, így a címek nem szabályokhoz vannak rendelve, teret adva a "több szabályt egy eszközhöz" lehetőségnek. A címlistában mindenképp szerepelniük kell, de szabálynak nem kötelező hozzárendelve lenni.
A legtöbb eszköz hajlamos váltogatni a RA-n kért IPv6 címét, Linux-on ezt a sysctl.conf módosításával lehet kikapcsolni, utána már a cím második része (EUI64) mindig ugyanaz lesz, azaz be lehet ide írni.
A sysctl.conf-ba ezt kell írni (a br0 nálam a helyi interfész neve):
net.ipv6.conf.default.addr_gen_mode = 0
net.ipv6.conf.br0.addr_gen_mode = 0


Ezután módosítsuk az alábbi sorokat:

IPv6 pool neve
:local poolname "digi"
Az interfész neve, ahova történik a címek osztása RA-val (fontos megadni, mert a pool és a használt interfész alapján lövi be a szkript a tartományt)
:local desiredif "br-lan"
PPPoE interfész neve
:local pppoeif "digi-pppoe"

Ha megvannak ezek is, lehet menteni. Ezzel a szkript a routeren van, elérhető, használható. Érdemes lehet kézi futtatással meggyőződni a működőképességéről.

Következő lépésként az IPv6-os tűzfal forward láncán engedjük a forgalmat, ahol a dst-address-list valamely korábban megadott eszközre mutat. Érdemes optimalizálni a szabályok sorrendjét, azaz a forward lánc drop-ja fölé rakni ezt mindenképp, de célszerű minél előrébb szerepeltetni.

Ezt követően a szkriptet csak rendeljük hozzá a PPPoE újracsatlakozáshoz, vagy - ha már van meglévő szkriptünk erre az esetre - írjuk bele, hogy futtassa ezt. Azért javaslom mindenképp új szkript felvételét, mert így a feladatok jól elkülöníthetőek.
A hozzárendelés alap esetben úgy történik, hogy a PPP profiles részen az on-up eseményhez kapcsoljuk a szkriptet.
A szkript a PPPoE újracsatlakozásnál gondoskodik a szabályok és a cím frissítéséről.

Remélem, hozzájárultam az IPv6 használatának terjedéséhez:D

<h2>English version</h2>
This easy-to-use script helps you to restart the IPv6 on your MikroTik, get a new prefix after PPPoE reconnect, and update neccessary firewal rules (for example allow incoming new connections to your home server).

<h3>WHY?</h3>
In my country Hungary, Magyar Telekom and DIGI provide IPv6 on their optical network (FTTB/FTTH) wiht PPPoE. MikroTik has very less support to work with dynamicaly changing IPv6 prefixes. For example, can't get new prefix or address automaticaly and can't update the firewall. The latest is important, if you have a home server, but you don't want to allow new incoming connections to other devices. It improves security, the devices can't accessible directly from outside (same as IPv4 NAT without port forwarding). By default, on most of the popular routers, the devices accessible from outside and usualy you can't change it.
</pre>
