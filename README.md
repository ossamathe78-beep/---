<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<title>Power Grid / شبكة الكهرباء</title>
<style>
:root{color-scheme:dark}
html,body{margin:0;height:100%;overflow:hidden;background:#141a16;font-family:system-ui,Tahoma,sans-serif;color:#e8efe9;user-select:none;-webkit-user-select:none}
#c{position:fixed;inset:0;width:100%;height:100%;display:block;touch-action:none}
#hud{position:fixed;top:env(safe-area-inset-top,0px);left:0;right:0;display:flex;gap:6px;justify-content:center;flex-wrap:wrap;padding:8px;pointer-events:none}
.chip{background:rgba(10,14,12,.82);border:1px solid #2c3a31;border-radius:10px;padding:5px 10px;font-size:14px;transition:color .3s}
#bar{position:fixed;bottom:0;left:0;right:0;display:flex;gap:6px;align-items:center;overflow-x:auto;padding:8px 8px calc(8px + env(safe-area-inset-bottom,0px));background:rgba(10,14,12,.9);border-top:1px solid #222c25}
button{background:#1f2a23;color:#e8efe9;border:1px solid #33433a;border-radius:10px;padding:8px 10px;font-size:14px;white-space:nowrap;font-family:inherit}
button:disabled{opacity:.4}
button.on{border-color:#7fd69a;background:#2b4a36}
#panel{position:fixed;left:50%;transform:translateX(-50%);bottom:76px;background:rgba(10,14,12,.96);border:1px solid #33433a;border-radius:12px;padding:10px;display:none;gap:8px;align-items:center;flex-wrap:wrap;justify-content:center;max-width:92%;font-size:14px;text-align:center}
#panel span{white-space:pre-line}
#toast{position:fixed;top:64px;left:50%;transform:translateX(-50%);background:#5a1f1f;color:#fff;padding:8px 14px;border-radius:10px;display:none;font-size:14px;pointer-events:none;max-width:92%;text-align:center;white-space:pre-line}
#pc,#co{position:fixed;top:0;bottom:64px;width:min(360px,94%);background:rgba(10,14,12,.97);overflow-y:auto;padding:12px;font-size:13px;display:none;z-index:5;box-sizing:border-box}
#pc{right:0;border-left:1px solid #33433a}
#co{left:0;border-right:1px solid #33433a;z-index:6}
#pc .r{border-bottom:1px solid #223;padding:8px 0}
#pc .m{color:#ffb4b4}
#co .px{font-size:28px;margin:8px 0}
#co .up{color:#7fd69a}
#co .dn{color:#ff6b6b}
#co .kv{display:flex;justify-content:space-between;gap:10px;padding:6px 0;border-bottom:1px solid #223}
#co h4{margin:14px 0 2px;font-size:13px;opacity:.8}
#chart{width:100%;height:auto;background:#0d1210;border-radius:8px;display:block}
</style>
</head>
<body>
<canvas id="c"></canvas>
<div id="hud"><div class="chip" id="money"></div><div class="chip" id="inc"></div><div class="chip" id="time"></div></div>
<div id="toast"></div>
<div id="panel"></div>
<div id="pc"></div>
<div id="co"></div>
<div id="bar"></div>
<script>
const cv=document.getElementById('c'),ctx=cv.getContext('2d');
const $=id=>document.getElementById(id);
const panel=$('panel'),bar=$('bar'),toastEl=$('toast'),pc=$('pc'),co=$('co');
let W,H;
function resize(){const d=devicePixelRatio||1;W=innerWidth;H=innerHeight;cv.width=W*d;cv.height=H*d;ctx.setTransform(d,0,0,d,0,0);}
addEventListener('resize',resize);resize();

/* ---------- language ---------- */
let lang='ar';
const LS={
ar:{shopT:'🛒 المتجر — اختار مبنى وحطو على الخريطة',shop:'🛒 المتجر',pcBtn:'💻 اللوحة',coBtn:'📊 الشركة',pcT:'لوحة التحكم',stop:'✖ وقّف',
 hint:'اسحب بين المباني للتوصيل · اسحب الأرض للتحريك',day:'نهار',night:'ليل',noM:'فلوس ناقصة',far:'بعيد! أقصى مسافة ',
 maxS:'المصدر عامر!',brd:'لازم قنطرة على النهر',wat:'ماشي على الماء',frs:'في الغابة ما كاينش بناء',busy:'مكان محجوز',
 zone:'مدينة مقفولة: ما تقدرش تبني فيها',
 opened:'تم فتح المدينة 🎉',locked:'مدينة مقفولة',open:'افتح',del:'حيّد',up:'طوّر',forest:'غابة',
 cab:'سلك',cabDel:'حيّد السلك',days:'يوم باقي',brk:'مكسور',
 cabNote:'كيتصلح بوحدو ملي يفسد (100 💰) + صيانة 2 💰 كل 20 ثانية للشبكة',
 frInfo:'غابة: كتولد خشب دايماً، وكل مصنع موصول بيها كياخد اللي يحتاجو. وصل خط بني لمصنع.',
 brownErr:'الخط البني كيمشي غير من غابة لمصنع',farF:'بعيد بزاف عن الغابة',perMin:'/ دقيقة',
 on:'موصولة',sun:'ضو',off:'مطفية',mine:'خدامة',noWood:'لا خشب!',noSrc:'لا مصدر!',ok:'✅ خدام',
 emp:'موظفين',work:'يشتغلو (كيخلصو)',rest:'كيرتاحو (نص الطاقة)',cust:'دور كتزور المقهى',
 grant:'🏛️ منحة طوارئ من المدينة: 200 💰',
 badLink:'ربط ماشي صحيح!\nغابة ← مصنع ← محطة ← موزع ← دار/مقهى\nشمس/بطارية ← محطة ← موزع ← دار/مقهى',
 d_nopow:'⚠️ ما كياخدش ضو: الموزع خاصو يكون مربوط بمحطة، والمحطة مربوطة بمصدر',
 d_dist:'⚠️ ما مربوطش بموزع: وصلو بموزع باش ياخد الضو',
 d_stn:'⚠️ ما مربوطش بالمحطة: الضو كيمشي عبر المحطة، وصلو بيها',
 d_low:'⚠️ الضو ناقص: زيد الطاقة (طوّر المحطة، زيد شمس أو مصنع) أو طوّر الموزع',
 d_over:'⚠️ الموزع مثقل: الديور كيحتاجو أكثر من سعتو، طوّرو',
 d_night:'🌙 الليل: الشمس ما كتخدمش، دير بطارية أو مصدر آخر',
 d_conn:'⚠️ ما مربوطش: اسحب منو خط لمبنى آخر باش يخدم',
 d_forest:'⚠️ المصنع ما عندو خشب: وصل خط بني من غابة',
 d_src:'⚠️ المحطة ما عندهاش مصدر: وصلها بشمس أو مصنع خشب أو بطارية',
 d_bat:'⚠️ البطارية خاوية: وصلها بالمحطة باش تشحن من الفائض',
 d_brk:'❌ السلك مكسور: غادي يتصلح بوحدو ملي يكون عندك 100 💰',
 coT:'🏢 شركة الشبكة',share:'سعر السهم',perf:'الأداء (آخر دقيقة)',rev:'الدخل / دقيقة',exp:'المصاريف / دقيقة',net:'الصافي / دقيقة',
 margin:'هامش الربح',profit:'ربح',loss:'خسارة',since:'منذ البداية',vsPrev:'مقارنة بالقراءة الأخيرة',
 staff:'الموظفون',staffWork:'يشتغلون الآن',blds:'المباني',houses:'الدور',happy:'سعادة السكان',avgSat:'نسبة الضو',
 evaded:'مرات التهرب من الدفع',lost:'فلوس ضايعة بالتهرب',wages:'الأجور',upk:'الصيانة',cabs:'الأسلاك',cafeNet:'ربح المقاهي',
 totRev:'مجموع الدخل',totExp:'مجموع المصاريف',totNet:'الصافي الكلي',val:'قيمة الشركة',cities:'المدن المفتوحة',costs:'تفصيل المصاريف'},
en:{shopT:'🛒 Shop — pick a building and place it on the map',shop:'🛒 Shop',pcBtn:'💻 Panel',coBtn:'📊 Company',pcT:'Control Panel',stop:'✖ Stop',
 hint:'Drag between buildings to connect · drag ground to move',day:'day',night:'night',noM:'Not enough money',far:'Too far! Max ',
 maxS:'Source full!',brd:'Needs a bridge over the river',wat:'Not on water',frs:'Cannot build in the forest',busy:'Spot taken',
 zone:'Locked city: you cannot build there',
 opened:'City opened 🎉',locked:'Locked city',open:'Open',del:'Remove',up:'Upgrade',forest:'Forest',
 cab:'Cable',cabDel:'Remove cable',days:'days left',brk:'Broken',
 cabNote:'Repairs itself when it wears out (100 💰) + 2 💰 upkeep every 20s for the network',
 frInfo:'Forest: produces wood forever; every factory linked to it takes what it needs. Link a brown line to a factory.',
 brownErr:'Brown lines only go from a forest to a factory',farF:'Too far from the forest',perMin:'/ min',
 on:'connected',sun:'power',off:'off',mine:'producing',noWood:'No wood!',noSrc:'No source!',ok:'✅ working',
 emp:'employees',work:'working (paid)',rest:'resting (half output)',cust:'houses visiting the café',
 grant:'🏛️ Emergency city grant: 200 💰',
 badLink:'Invalid link!\nForest → Wood plant → Station → Distributor → House/Café\nSolar/Battery → Station → Distributor → House/Café',
 d_nopow:'⚠️ No power: its distributor must be linked to a station, and the station to a source',
 d_dist:'⚠️ Not linked to a distributor: connect it to one to receive power',
 d_stn:'⚠️ Not linked to a station: power flows through the station, link it',
 d_low:'⚠️ Low power: add capacity (upgrade the station, add solar/plants) or upgrade the distributor',
 d_over:'⚠️ Distributor overloaded: houses need more than its capacity, upgrade it',
 d_night:'🌙 Night: solar is off, add a battery or another source',
 d_conn:'⚠️ Not connected: drag a line from it to another building',
 d_forest:'⚠️ No wood: link a brown line from a forest',
 d_src:'⚠️ Station has no source: link it to solar, a wood plant or a battery',
 d_bat:'⚠️ Battery empty: link it to the station to charge from surplus',
 d_brk:'❌ Cable broken: it will repair itself once you have 100 💰',
 coT:'🏢 Grid Corp',share:'Share price',perf:'Performance (last minute)',rev:'Revenue / min',exp:'Expenses / min',net:'Net / min',
 margin:'Profit margin',profit:'Profit',loss:'Loss',since:'Since start',vsPrev:'vs last reading',
 staff:'Employees',staffWork:'working now',blds:'Buildings',houses:'Houses',happy:'Resident happiness',avgSat:'Power level',
 evaded:'Payments evaded',lost:'Money lost to evasion',wages:'Wages',upk:'Upkeep',cabs:'Cables',cafeNet:'Café profit',
 totRev:'Total revenue',totExp:'Total expenses',totNet:'Total net',val:'Company value',cities:'Cities opened',costs:'Expense breakdown'}
};
const t=k=>LS[lang][k]||k;

/* ---------- constants ---------- */
const DAY=60,LIFE=20*DAY,MAXLV=3,CH=400,RES=100,FZ=600,RFC_CELL=1000,HALF=200,FOREST_MIN=450;
// city prices are 5x the old ones
const COSTS=[10000,17500,25000,37500,55000],BCAP=600,SELL=0.02,PAY_EVERY=6,SRC=new Set(['station','solar','factory']);
const CAB_FIX=100,CAB_FEE=2,CAB_EVERY=20,DIST_CAP=300;
// staff: every power building needs EMP employees, paid WAGE per employee per minute while working;
// they work SHIFT seconds, then rest SHIFT seconds (no wage, building runs at half output)
const EMP=10,WAGE=10,SHIFT=60,CAFE_R=260;
// c=buy price, u=upgrade base (cost = u*level), k=upkeep per minute per level (only for bought buildings)
const TY={
 station:{n:['محطة رئيسية','Main Station'],i:'⚡',c:600,u:500,k:1},
 solar:{n:['شمس','Solar'],i:'☀️',c:480,u:400,k:.5},
 factory:{n:['مصنع خشب','Wood Plant'],i:'🏭',c:300,u:250,k:.5},
 dist:{n:['موزع','Distributor'],i:'🏢',c:120,u:100,k:.5},
 accel:{n:['مسرع','Accelerator'],i:'🚀',c:240,u:150,k:1},
 battery:{n:['بطارية','Battery'],i:'🔋',c:180,u:150,k:.5},
 cafe:{n:['مقهى وحلويات','Café & Sweets'],i:'☕',c:250,u:200,k:.5},
 bridge:{n:['قنطرة','Bridge'],i:'🌉',c:120,u:0,k:0},
 house:{n:['دار','House'],i:'🏠',c:0,u:300,k:0},
 rfac:{n:['مصنع بعيد','Remote Plant'],i:'🏗️',c:0,u:400,k:0}
};
const STAFF=new Set(['station','solar','factory','dist','accel','battery']);
// allowed links: forest>factory>station>distributor>house/cafe ; solar>battery>station
const LINK={
 station:['factory','solar','battery','dist','accel'],
 factory:['station','accel'],
 solar:['station','battery','accel'],
 battery:['station','solar','accel'],
 dist:['station','house','rfac','cafe','accel','dist'],
 accel:['station','factory','solar','battery','dist','accel'],
 house:['dist'],
 rfac:['dist'],
 cafe:['dist']
};
const nm=type=>TY[type].n[lang==='ar'?0:1];
const isCons=n=>n.type==='house'||n.type==='rfac'||n.type==='cafe';
const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));
const hasStaff=n=>STAFF.has(n.type)||n.type==='cafe';
const working=n=>((GT+n.ph)%(SHIFT*2))<SHIFT;
const eff=n=>(hasStaff(n)&&!working(n))?0.5:1;
// area of a city: the 400x400 block plus the shop above it
const inZone=(x,y,c,p)=>Math.abs(x-c.x)<HALF+p&&y-c.y>-(300+p)&&y-c.y<HALF+p;

/* ---------- state ---------- */
let money=4000,GT=0,placing=null,gDay=true,openedCount=1,nid=1,lastDt=0,lastMoney=4000,pcOpen=false,coOpen=false,pcT=0,coT=0,lastGrant=-999,cabT=0;
const NB=new Map(),E=[],stores=[],bridges=[],cities=[],fx=[],walkers=[],RFS=new Set();
const cam={x:0,y:0,s:.6};let mouse=null,walkT=0;
// company statistics
const stat={rev:0,exp:0,wages:0,upk:0,cab:0,cafe:0,evaded:0,lost:0};
const rates={rev:0,exp:0,net:0};
let statT=0,lastRev=0,lastExp=0,price=100;const samples=[],priceHist=[100];
const earn=x=>{money+=x;stat.rev+=x;};
const pay=(x,cat)=>{money=Math.max(0,money-x);stat.exp+=x;if(cat)stat[cat]+=x;};

/* ---------- terrain ---------- */
let SEED=Math.floor(Math.random()*1e9);
function hash(x,y){let h=Math.imul(x|0,374761393)^Math.imul(y|0,668265263)^Math.imul(SEED,982451653);h=Math.imul(h^(h>>>13),1274126177);h^=h>>>16;return (h>>>0)/4294967295;}
function vnoise(x,y){const xi=Math.floor(x),yi=Math.floor(y),xf=x-xi,yf=y-yi;const u=xf*xf*(3-2*xf),v=yf*yf*(3-2*yf);
 const a=hash(xi,yi),b=hash(xi+1,yi),c=hash(xi,yi+1),d=hash(xi+1,yi+1);
 return a+(b-a)*u+(c-a)*v+(a-b-c+d)*u*v;}
function fbm(x,y){return vnoise(x,y)*.65+vnoise(x*2.1+7,y*2.1+3)*.35;}
// rivers: thin ribbons in some regions, plus one winding river passing near each city (never inside it)
const baseWater=(x,y)=>Math.abs(fbm(x/260,y/260)-.5)<.008&&vnoise(x/900+40,y/900+9)>.55;
function makeRiver(){const a=Math.random()*Math.PI*2;return{cos:Math.cos(a),sin:Math.sin(a),ph:Math.random()*6.28};}
function cityRiver(x,y){
 for(const c of cities){const r=c.river;if(!r)continue;
  const dx=x-c.x,dy=y-c.y;
  if(Math.abs(dx)>1100||Math.abs(dy)>1100)continue;
  const n=dx*r.cos+dy*r.sin,u=-dx*r.sin+dy*r.cos;
  if(Math.abs(u)<750&&Math.abs(n-340-22*Math.sin(u/90+r.ph))<9)return true;}
 return false;}
const isWater=(x,y)=>{
 for(const c of cities)if(inZone(x,y,c,40))return false;   // no water inside or right next to a city
 return baseWater(x,y)||cityRiver(x,y);};
const isHill=(x,y)=>!isWater(x,y)&&fbm(x/420+50,y/420+50)>.64;

const CC=new Map();
function chunkCanvas(cx,cy){const k=cx+','+cy;if(CC.has(k))return CC.get(k);
 const c=document.createElement('canvas');c.width=c.height=RES;const g=c.getContext('2d');const im=g.createImageData(RES,RES);
 for(let j=0;j<RES;j++)for(let i=0;i<RES;i++){const x=cx*CH+(i+.5)*CH/RES,y=cy*CH+(j+.5)*CH/RES;
  const col=isWater(x,y)?[44,104,156]:isHill(x,y)?[40,54,44]:[26,36,30];
  const o=(j*RES+i)*4;im.data[o]=col[0];im.data[o+1]=col[1];im.data[o+2]=col[2];im.data[o+3]=255;}
 g.putImageData(im,0,0);CC.set(k,c);return c;}

const FC=new Map();
function forestsIn(cx,cy){const k=cx+','+cy;if(FC.has(k))return FC.get(k);const arr=[];
 if(hash(cx*31+7,cy*17+3)<.5){const r=70+hash(cx+101,cy+37)*40;
  const f={id:'f'+cx+'_'+cy,x:cx*FZ+FZ*(.2+.6*hash(cx+5,cy+9)),y:cy*FZ+FZ*(.2+.6*hash(cx+71,cy+13)),r,trees:[]};
  for(let q=0;q<22;q++){const a=q*2.4,rr=r*.8*Math.sqrt((q+1)/22);f.trees.push([Math.cos(a)*rr,Math.sin(a)*rr]);}
  arr.push(f);}
 FC.set(k,arr);return arr;}
// forests never appear close to a city, so far cities depend on solar power
const validForest=f=>cities.every(c=>Math.hypot(c.x-f.x,c.y-f.y)>=FOREST_MIN);
function forestsRect(x0,y0,x1,y1){const out=[];
 for(let cx=Math.floor(x0/FZ)-1;cx<=Math.floor(x1/FZ)+1;cx++)for(let cy=Math.floor(y0/FZ)-1;cy<=Math.floor(y1/FZ)+1;cy++)
  out.push(...forestsIn(cx,cy).filter(validForest));
 return out;}
function getForest(id){const [a,b]=id.slice(1).split('_').map(Number);const f=forestsIn(a,b)[0];return f&&validForest(f)?f:undefined;}
function forestBlock(x,y){return forestsRect(x-150,y-150,x+150,y+150).some(f=>Math.hypot(x-f.x,y-f.y)<f.r+10);}
function rawForestNear(x,y){
 for(let cx=Math.floor((x-FOREST_MIN-60)/FZ);cx<=Math.floor((x+FOREST_MIN+60)/FZ);cx++)
  for(let cy=Math.floor((y-FOREST_MIN-60)/FZ);cy<=Math.floor((y+FOREST_MIN+60)/FZ);cy++)
   for(const f of forestsIn(cx,cy))if(Math.hypot(f.x-x,f.y-y)<FOREST_MIN+60)return true;
 return false;}

/* ---------- buildings & cities ---------- */
function nextCost(n){return n<COSTS.length?COSTS[n]:Math.round(COSTS[4]*1.5**(n-4));}
function addB(type,x,y,paid=0,auto=false){const id=nid++;
 NB.set(id,{id,type,x,y,lv:1,st:0,sat:0,deg:0,noSrc:false,wood:false,hasHub:false,live:false,over:false,need:0,paid,auto,tm:0,
  ph:Math.random()*SHIFT*2,happy:.7,cafe:false,cust:0});return id;}
function buildCityContents(c){
 for(let i=0;i<4;i++)for(let j=0;j<3;j++)addB('house',c.x-120+i*80,c.y-80+j*80);
 stores.push({x:c.x,y:c.y-280});}
// a new city may only appear on empty ground: no forests, buildings, shops or bridges under it
function cityFree(x,y){
 if(cities.some(o=>Math.max(Math.abs(o.x-x),Math.abs(o.y-y))<=520))return false;
 const hit=o=>Math.abs(o.x-x)<HALF+120&&o.y-y>-(300+120)&&o.y-y<HALF+120;
 for(const b of NB.values())if(hit(b))return false;
 for(const s of stores)if(hit(s))return false;
 for(const b of bridges)if(hit(b))return false;
 if(rawForestNear(x,y))return false;
 return true;}
function spawnNear(par){
 for(let q=0;q<400;q++){
  const a=Math.random()*Math.PI*2,d=q<150?800:800+(q-149)*8;
  const x=par.x+Math.cos(a)*d,y=par.y+Math.sin(a)*d;
  if(cityFree(x,y)){const c={x,y,open:false,par,river:makeRiver()};cities.push(c);CC.clear();return c;}}
 return null;}
// remote plants appear automatically far from cities; they cannot be bought or built
function ensureRF(cx,cy){const k=cx+','+cy;if(RFS.has(k))return;RFS.add(k);
 if(hash(cx*13+1,cy*7+9)>.4)return;
 const x=cx*RFC_CELL+RFC_CELL*(.2+.6*hash(cx+3,cy+5)),y=cy*RFC_CELL+RFC_CELL*(.2+.6*hash(cx+8,cy+2));
 if(Math.hypot(x,y)<600||isWater(x,y))return;
 if(cities.some(c=>Math.max(Math.abs(c.x-x),Math.abs(c.y-y))<400))return;
 if(forestBlock(x,y))return;
 addB('rfac',x,y,0,true);}
const c0={x:0,y:0,open:true,par:null,river:makeRiver()};cities.push(c0);buildCityContents(c0);spawnNear(c0);

/* ---------- coords ---------- */
const toS=(x,y)=>[W/2+(x-cam.x)*cam.s,H/2+(y-cam.y)*cam.s];
const toW=(px,py)=>[cam.x+(px-W/2)/cam.s,cam.y+(py-H/2)/cam.s];
function zoomAt(px,py,f){const [wx,wy]=toW(px,py);cam.s=Math.min(4,Math.max(.12,cam.s*f));cam.x=wx-(px-W/2)/cam.s;cam.y=wy-(py-H/2)/cam.s;}

/* ---------- graph ---------- */
function node(id){
 if(typeof id==='string'){const f=getForest(id);return f?{type:'forest',x:f.x,y:f.y,r:f.r,id:f.id}:null;}
 return NB.get(id)||null;}
function crossOK(x1,y1,x2,y2){
 const d=Math.hypot(x2-x1,y2-y1);if(d===0)return true;const n=Math.max(1,Math.ceil(d/8));
 for(let i=0;i<=n;i++){const x=x1+(x2-x1)*i/n,y=y1+(y2-y1)*i/n;
  if(isWater(x,y)&&!bridges.some(b=>Math.hypot(b.x-x,b.y-y)<25))return false;}
 return true;}
function allPoints(){return [...NB.values(),...stores,...bridges];}
function canPlace(type,x,y){
 if(cities.some(c=>!c.open&&inZone(x,y,c,10)))return t('zone');
 if(type==='bridge'){if(!isWater(x,y))return t('wat');}
 else if(isWater(x,y))return t('wat');
 if(forestBlock(x,y))return t('frs');
 if(allPoints().some(o=>Math.hypot(o.x-x,o.y-y)<40))return t('busy');
 return null;}
const costAt=(type,x,y)=>Math.round(TY[type].c*(isHill(x,y)?1.5:1));
const reach=n=>n.type==='accel'?200+300*n.lv:n.type==='dist'?200+100*(n.lv-1):n.type==='station'?300+100*(n.lv-1):200;

function connect(a,b){
 if(a===b)return;
 const fa=typeof a==='string',fb=typeof b==='string';
 if(fa||fb){const fid=fa?a:b,bid=fa?b:a,f=node(fid),fac=NB.get(bid);
  if(!f||!fac||fac.type!=='factory'){toast(t('brownErr'));return;}
  if(Math.hypot(f.x-fac.x,f.y-fac.y)>f.r+200){toast(t('farF'));return;}
  if(!crossOK(f.x,f.y,fac.x,fac.y)){toast(t('brd'));return;}
  if(E.some(e=>(e.a===fid&&e.b===bid)||(e.a===bid&&e.b===fid)))return;
  E.push({a:fid,b:bid,brown:true,born:GT,broken:false,ph:0,live:false});return;}
 const A=NB.get(a),B=NB.get(b);if(!A||!B)return;
 if(E.some(e=>!e.brown&&((e.a===a&&e.b===b)||(e.a===b&&e.b===a))))return;
 if(!LINK[A.type]||!LINK[A.type].includes(B.type)){toast(t('badLink'));return;}
 const R=Math.max(reach(A),reach(B));
 if(Math.hypot(A.x-B.x,A.y-B.y)>R){toast(`${t('far')}${R}`);return;}
 if(!crossOK(A.x,A.y,B.x,B.y)){toast(t('brd'));return;}
 for(const n of [A,B])if(SRC.has(n.type)&&E.filter(e=>!e.brown&&(e.a===n.id||e.b===n.id)).length>=20){toast(t('maxS'));return;}
 E.push({a,b,brown:false,born:GT,broken:false,ph:0,live:false});}

function removeB(id){const b=NB.get(id);if(!b)return;const r=Math.round(b.paid*.8);money+=r;NB.delete(id);
 for(let i=E.length-1;i>=0;i--)if(E[i].a===id||E[i].b===id)E.splice(i,1);
 closePanel();toast(`+${r} 💰`);}
function upg(id){const b=NB.get(id),c=TY[b.type].u*b.lv;
 if(money<c){toast(t('noM'));return;}money-=c;b.lv++;showBld(id);toast(`Lv${b.lv} ✔`);}

/* ---------- simulation ---------- */
function sim(dt){
 // cables wear out after 20 days and repair themselves for 100 when money allows
 for(const e of E){
  if(!e.broken&&GT-e.born>=LIFE)e.broken=true;
  if(e.broken&&money>=CAB_FIX){pay(CAB_FIX,'cab');e.broken=false;e.born=GT;
   const A=node(e.a),B=node(e.b);if(A&&B)fx.push({x:(A.x+B.x)/2,y:(A.y+B.y)/2,t:0,s:`🔧 -${CAB_FIX}💰`});}}
 // network upkeep: 2 coins every 20 seconds
 cabT+=dt;if(cabT>=CAB_EVERY){cabT-=CAB_EVERY;if(E.length)pay(CAB_FEE,'cab');}

 // cafés: which houses are close to a working café, and how many customers each café has
 const cafes=[];for(const b of NB.values())if(b.type==='cafe'){b.cust=0;if(b.sat>0)cafes.push(b);}
 for(const b of NB.values())if(b.type==='house'){
  b.cafe=false;
  for(const c of cafes)if(Math.hypot(c.x-b.x,c.y-b.y)<CAFE_R){b.cafe=true;if(b.sat>.3)c.cust++;}
  // happiness: power level, café nearby, and the high cost of a level-3 house
  b.happy=clamp(.35+.35*b.sat+(b.cafe?.3:0)-(b.lv>=3?.1:0),0,1);}

 const wood=new Set();
 for(const e of E)if(e.brown&&!e.broken){const fa=typeof e.a==='string';if(getForest(fa?e.a:e.b))wood.add(fa?e.b:e.a);}
 const adj=new Map();for(const id of NB.keys())adj.set(id,[]);
 for(const e of E)if(!e.brown&&!e.broken){adj.get(e.a).push(e.b);adj.get(e.b).push(e.a);}
 for(const b of NB.values()){b.deg=adj.get(b.id).length;b.over=false;if(b.type==='factory')b.wood=wood.has(b.id);}
 const seen=new Set();
 for(const id of NB.keys()){
  if(seen.has(id))continue;
  const comp=[],stk=[id];seen.add(id);
  while(stk.length){const c=NB.get(stk.pop());comp.push(c);for(const m of adj.get(c.id))if(!seen.has(m)){seen.add(m);stk.push(m);}}

  const stations=comp.filter(n=>n.type==='station');
  const fed=new Set(),reachD=new Set();
  const walk=(allowed,set)=>{
   const vis=new Set(stations.map(s=>s.id)),q=[...stations];
   while(q.length){const c=q.pop();
    for(const m of adj.get(c.id)){if(vis.has(m))continue;const n=NB.get(m);
     if(!allowed.includes(n.type))continue;vis.add(m);set.add(m);q.push(n);}}};
  walk(['solar','factory','battery','accel'],fed);
  walk(['dist','accel'],reachD);

  let gen=0,hub=0,sto=0;const bats=[],dead=[],groups=new Map();
  for(const n of comp){const L=n.lv;
   if(isCons(n)){
    const dm=adj.get(n.id).find(m=>reachD.has(m)&&NB.get(m).type==='dist');
    if(dm!==undefined){
     let g=groups.get(dm);if(!g){g={d:NB.get(dm),cons:[],dem:0};groups.set(dm,g);}
     n.need=n.type==='house'?20*L:n.type==='cafe'?30*L:100*L;g.cons.push(n);g.dem+=n.need;}
    else dead.push(n);}
   else if(n.type==='station')hub+=100*L*eff(n);
   else if(n.type==='solar'){if(fed.has(n.id))gen+=gDay?40*L*eff(n):0;}
   else if(n.type==='factory'){if(n.wood&&fed.has(n.id))gen+=6*L*eff(n);}
   else if(n.type==='battery'){if(fed.has(n.id)){bats.push(n);sto+=n.st;}}}

  let demTotal=0;
  for(const g of groups.values()){
   g.cap=DIST_CAP*g.d.lv*eff(g.d);g.pass=Math.min(g.dem,g.cap);g.ratio=g.dem>0?g.pass/g.dem:0;
   g.d.over=g.dem>g.cap;demTotal+=g.pass;}
  const hasStation=stations.length>0;
  const use=hasStation?Math.min(gen,hub):0;
  const net=use-demTotal;let f=1,sold=0;
  if(net>=0){
   let rem=net*dt;
   const open=bats.filter(x=>x.st<BCAP*x.lv);
   if(open.length){const share=rem/open.length;let left=0;
    for(const x of open){const add=Math.min(share,BCAP*x.lv-x.st);x.st+=add;left+=share-add;}
    rem=left;}
   sold=(hasStation&&demTotal>0)?rem*SELL:0;
  }else{
   const need=-net*dt;
   const limit=hasStation?Math.max(0,hub-use)*dt:0;
   const taken=Math.min(need,limit,sto);let left=taken;
   for(const x of bats){if(left<=0)break;const q=Math.min(x.st,left);x.st-=q;left-=q;}
   f=Math.min(1,(use*dt+taken)/(demTotal*dt));
  }
  const served=new Set();
  for(const g of groups.values()){
   const s=f*g.ratio;g.d.sat=s;
   for(const n of g.cons){n.sat=s;served.add(n);}}
  for(const n of comp){
   if(!isCons(n)&&n.type!=='dist')n.sat=f;
   n.noSrc=gen===0&&sto===0;
   n.hasHub=n.type==='station'||fed.has(n.id)||reachD.has(n.id)||served.has(n);
   n.live=n.hasHub&&use>0;}
  for(const n of dead){n.sat=0;n.live=false;n.hasHub=false;}

  // payouts every 6 seconds, only for served buildings
  for(const n of served){
   const L=n.lv;
   n.tm+=dt;
   if(n.tm<PAY_EVERY)continue;
   n.tm-=PAY_EVERY;
   if(n.sat<=0)continue;
   if(n.type==='cafe'){
    // café: +2 per employee, -1 per employee; it needs customers from nearby houses
    const ratio=Math.min(1,n.cust/4),gain=EMP*2*L*n.sat*ratio*eff(n),cost=EMP*1;
    earn(gain);pay(cost);stat.cafe+=gain-cost;
    const d=gain-cost;
    fx.push({x:n.x,y:n.y,t:0,s:`${d>=0?'+':''}${d.toFixed(0)}💰`});
   }else if(n.type==='rfac'){
    const amt=Math.floor(12*L*n.sat);if(amt>0){earn(amt);fx.push({x:n.x,y:n.y,t:0,s:`+${amt}💰`});}
   }else{
    // houses: some refuse to pay when power is short, the house is level 3 (expensive) or people are unhappy
    const base=10*L*n.sat;
    const p=clamp((1-n.sat)*.6+(L>=3?.2:0)+(1-n.happy)*.3,0,.9);
    if(Math.random()<p){stat.evaded++;stat.lost+=Math.floor(base);fx.push({x:n.x,y:n.y,t:0,s:'🙈'});}
    else{let amt=Math.floor(base);if(n.cafe)amt=Math.max(0,amt-1);   // the café visit costs residents 1 coin
     if(amt>0){earn(amt);fx.push({x:n.x,y:n.y,t:0,s:`+${amt}💰`});}}
   }
  }
  if(sold>0){earn(sold);
   if(Math.random()<dt*.3){const h=stations[0];if(h)fx.push({x:h.x,y:h.y,t:0,s:`+${sold.toFixed(1)}💰`});}}
 }
 // wages: only while employees are working (they rest the other minute and cost nothing)
 for(const b of NB.values())if(STAFF.has(b.type)&&working(b))pay(EMP*WAGE/60*dt,'wages');
 // upkeep only for buildings the player bought
 let U=0;for(const b of NB.values())if(!b.auto)U+=TY[b.type].k*b.lv;
 pay(U/60*dt,'upk');
 for(const e of E){
  if(e.broken){e.live=false;continue;}
  if(e.brown){const f=NB.get(typeof e.a==='string'?e.b:e.a);e.live=!!(f&&f.wood&&f.hasHub);}
  else{const A=NB.get(e.a),B=NB.get(e.b);e.live=!!(A&&B&&A.live&&B.live);}
  if(e.live)e.ph=(e.ph+dt*.5)%1;
 }}

/* ---------- company: rates and share price (every 5 seconds) ---------- */
function sample(){
 samples.push({r:stat.rev-lastRev,e:stat.exp-lastExp});lastRev=stat.rev;lastExp=stat.exp;
 if(samples.length>12)samples.shift();
 const f=60/(samples.length*5);let r=0,e=0;for(const s of samples){r+=s.r;e+=s.e;}
 rates.rev=r*f;rates.exp=e*f;rates.net=rates.rev-rates.exp;
 let hs=0,sa=0,ha=0;for(const b of NB.values())if(b.type==='house'){hs++;sa+=b.sat;ha+=b.happy;}
 const avgS=hs?sa/hs:0,avgH=hs?ha/hs:0;
 // the price follows profit, power level and happiness, with a little market noise
 const target=100+rates.net*.05+(avgS-.5)*20+(avgH-.6)*20;
 price=Math.max(1,price+(target-price)*.12+(Math.random()-.5)*2.5);
 priceHist.push(price);if(priceHist.length>120)priceHist.shift();}

/* ---------- diagnostics ---------- */
function diag(b){
 if(isCons(b)){
  if(b.deg===0)return 'd_dist';
  if(!b.hasHub)return 'd_nopow';
  return b.sat===0?'d_nopow':b.sat<.99?'d_low':null;}
 if(b.type==='factory'){if(!b.wood)return 'd_forest';return b.hasHub?null:'d_stn';}
 if(b.type==='solar'){if(!gDay)return 'd_night';if(b.deg===0)return 'd_conn';return b.hasHub?null:'d_stn';}
 if(b.type==='station')return b.deg===0?'d_conn':b.noSrc?'d_src':null;
 if(b.type==='accel')return b.deg===0?'d_conn':null;
 if(b.type==='dist')return b.deg===0?'d_conn':!b.hasHub?'d_stn':b.over?'d_over':null;
 if(b.type==='battery'){if(b.deg===0)return 'd_conn';if(!b.hasHub)return 'd_stn';return b.noSrc?'d_bat':null;}
 return null;}

/* ---------- drawing ---------- */
function icon(sx,sy,em,r,col){
 ctx.beginPath();ctx.arc(sx,sy,r,0,Math.PI*2);ctx.fillStyle='#0d1210';ctx.fill();
 ctx.lineWidth=2;ctx.strokeStyle=col;ctx.stroke();
 ctx.font=`${r*1.1}px sans-serif`;ctx.textAlign='center';ctx.textBaseline='middle';ctx.fillStyle='#fff';ctx.fillText(em,sx,sy+1);}

function drawAll(){
 const s=cam.s;
 ctx.globalAlpha=1;ctx.fillStyle='#141a16';ctx.fillRect(0,0,W,H);
 const x0=cam.x-W/2/s,x1=cam.x+W/2/s,y0=cam.y-H/2/s,y1=cam.y+H/2/s;
 for(let cx=Math.floor(x0/CH);cx<=Math.floor(x1/CH);cx++)
  for(let cy=Math.floor(y0/CH);cy<=Math.floor(y1/CH);cy++){
   const [px,py]=toS(cx*CH,cy*CH);ctx.drawImage(chunkCanvas(cx,cy),px,py,CH*s+1,CH*s+1);}
 for(let cx=Math.floor(x0/RFC_CELL);cx<=Math.floor(x1/RFC_CELL);cx++)
  for(let cy=Math.floor(y0/RFC_CELL);cy<=Math.floor(y1/RFC_CELL);cy++)ensureRF(cx,cy);

 ctx.strokeStyle='#5d625e';ctx.lineWidth=Math.max(1.5,8*s);ctx.lineCap='round';
 for(const c of cities)if(c.par){const [a,b]=toS(c.x,c.y),[p,q]=toS(c.par.x,c.par.y);
  ctx.beginPath();ctx.moveTo(p,q);ctx.lineTo(a,b);ctx.stroke();}

 for(const f of forestsRect(x0,y0,x1,y1)){
  const [sx,sy]=toS(f.x,f.y);ctx.fillStyle='#17361f';ctx.beginPath();ctx.arc(sx,sy,f.r*s,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='#2c6a3a';
  for(const [dx,dy] of f.trees){ctx.beginPath();ctx.arc(sx+dx*s,sy+dy*s,Math.max(2,9*s),0,Math.PI*2);ctx.fill();}}

 for(const c of cities){
  const [sx,sy]=toS(c.x-HALF,c.y-HALF),q=2*HALF*s;
  ctx.fillStyle=c.open?'rgba(120,170,130,.05)':'rgba(0,0,0,.3)';ctx.fillRect(sx,sy,q,q);
  ctx.strokeStyle=c.open?'#4f7a5c':'#7a8580';ctx.lineWidth=2;ctx.strokeRect(sx,sy,q,q);
  if(!c.open){ctx.fillStyle='#e8efe9';ctx.font='14px sans-serif';ctx.textAlign='center';ctx.textBaseline='middle';
   ctx.fillText(`🔒 ${nextCost(openedCount-1)} 💰`,sx+q/2,sy+q/2);}
 }

 for(const e of E){const A=node(e.a),B=node(e.b);if(!A||!B)continue;
  const [p,q]=toS(A.x,A.y),[r,tt]=toS(B.x,B.y);
  ctx.strokeStyle=e.broken?'#ff4d4d':e.brown?'#9a6a3a':'#3ea0ff';ctx.lineWidth=e.broken?2:e.brown?3:2.5;
  ctx.beginPath();ctx.moveTo(p,q);ctx.lineTo(r,tt);ctx.stroke();
  if(e.live){ctx.fillStyle='#d7f0ff';for(const k of [e.ph,(e.ph+.5)%1]){
   ctx.beginPath();ctx.arc(p+(r-p)*k,q+(tt-q)*k,2.2,0,Math.PI*2);ctx.fill();}}
 }

 for(const br of bridges){const [sx,sy]=toS(br.x,br.y);icon(sx,sy,'🌉',9,'#8a5a2c');}
 for(const st of stores){const [sx,sy]=toS(st.x,st.y);icon(sx,sy,'🛒',13,'#ffb84d');}

 for(const b of NB.values()){
  const [sx,sy]=toS(b.x,b.y);const isH=b.type==='house';
  const r=isH?15:12+b.lv;
  let col='#7fd69a';
  if(isH){col=b.sat>.99?'#ffd166':b.sat>0?'#b08a4a':'#4a4f4c';
   if(b.sat>0){ctx.globalAlpha=.25+.15*Math.sin(GT*3+b.x);ctx.beginPath();ctx.arc(sx,sy,r+6,0,Math.PI*2);ctx.fillStyle='#ffd166';ctx.fill();ctx.globalAlpha=1;}}
  else if(b.type==='rfac'||b.type==='cafe')col=b.sat>.99?'#ffd166':b.sat>0?'#b08a4a':'#4a4f4c';
  else if(b.type==='factory')col=b.wood?(b.hasHub?'#9a6a3a':'#777'):'#ff6b6b';
  else if(b.type==='dist')col=b.deg===0||!b.hasHub?'#777':b.over?'#ff9d4d':'#7fd69a';
  else if(b.type==='station'||b.type==='battery'||b.type==='solar')col=(b.deg===0||(b.type!=='station'&&!b.hasHub))?'#777':'#7fd69a';
  icon(sx,sy,TY[b.type].i,r,col);
  let tag='',bad=false;
  if(b.type==='factory'&&!b.wood){tag=t('noWood');bad=true;}
  else if(b.type==='battery'){tag=Math.round(b.st/(BCAP*b.lv)*100)+'%';}
  else if(b.type==='rfac'||b.type==='house'||b.type==='cafe')tag=b.sat>0?Math.round(b.sat*100)+'%':'';
  else if(b.type==='dist'&&b.over){tag='⚠';bad=true;}
  if(b.lv>1)tag=`Lv${b.lv} ${tag}`;
  if(hasStaff(b)&&!working(b))tag+=' 💤';
  if(tag){ctx.font='11px sans-serif';ctx.textAlign='center';ctx.textBaseline='top';ctx.fillStyle=bad?'#ff8a8a':'#cfe';ctx.fillText(tag,sx,sy+r+3);}
 }

 // residents walking from houses to cafés
 for(let i=walkers.length-1;i>=0;i--){const w=walkers[i];w.t+=lastDt/2.5;
  if(w.t>=1||!NB.has(w.a.id)||!NB.has(w.b.id)){walkers.splice(i,1);continue;}
  const [sx,sy]=toS(w.a.x+(w.b.x-w.a.x)*w.t,w.a.y+(w.b.y-w.a.y)*w.t);
  ctx.font='12px sans-serif';ctx.textAlign='center';ctx.textBaseline='middle';ctx.fillStyle='#fff';ctx.fillText('🚶',sx,sy);}

 for(const f of fx){f.t+=lastDt;const [sx,sy]=toS(f.x,f.y-f.t*30);
  ctx.globalAlpha=Math.max(0,1-f.t/1.2);ctx.fillStyle='#ffd166';ctx.font='12px sans-serif';ctx.textAlign='center';ctx.textBaseline='middle';ctx.fillText(f.s,sx,sy);}
 ctx.globalAlpha=1;
 for(let i=fx.length-1;i>=0;i--)if(fx[i].t>1.2)fx.splice(i,1);

 if(placing&&mouse){const [wx,wy]=toW(mouse.x,mouse.y);const [sx,sy]=toS(wx,wy);
  const err=canPlace(placing,wx,wy);ctx.globalAlpha=.6;icon(sx,sy,TY[placing].i,12,err?'#ff6b6b':'#7fd69a');ctx.globalAlpha=1;}

 if(drag&&drag.link&&drag.moved){const A=node(drag.link);if(A){const [p,q]=toS(A.x,A.y);
  ctx.strokeStyle=typeof drag.link==='string'?'#9a6a3a':'#3ea0ff';ctx.lineWidth=2;ctx.setLineDash([6,4]);
  ctx.beginPath();ctx.moveTo(p,q);ctx.lineTo(drag.px,drag.py);ctx.stroke();ctx.setLineDash([]);}}
}

/* ---------- UI ---------- */
let tt;
function toast(m){toastEl.textContent=m;toastEl.style.display='block';clearTimeout(tt);tt=setTimeout(()=>toastEl.style.display='none',2600);}
function showP(text,btns=[]){
 panel.innerHTML='';const s=document.createElement('span');s.textContent=text;panel.appendChild(s);
 for(const [tx,fn,dis] of btns){const b=document.createElement('button');b.textContent=tx;b.disabled=!!dis;b.onclick=fn;panel.appendChild(b);}
 const x=document.createElement('button');x.textContent='✖';x.onclick=closePanel;panel.appendChild(x);
 panel.style.display='flex';}
function closePanel(){panel.style.display='none';}
function openShop(){showP(t('shopT'),['station','solar','factory','dist','accel','battery','cafe','bridge'].map(k=>[`${TY[k].i} ${nm(k)} ${TY[k].c}💰`,()=>setPlace(k)]));}
function setPlace(k){placing=k;closePanel();updateBar();}
function toggleLang(){lang=lang==='ar'?'en':'ar';document.documentElement.lang=lang;document.documentElement.dir=lang==='ar'?'rtl':'ltr';updateBar();closePanel();if(pcOpen)renderPC();if(coOpen)renderCo();}
function togglePC(){pcOpen=!pcOpen;pc.style.display=pcOpen?'block':'none';if(pcOpen){coOpen=false;co.style.display='none';renderPC();pcT=GT;}}
function toggleCo(){coOpen=!coOpen;co.style.display=coOpen?'block':'none';if(coOpen){pcOpen=false;pc.style.display='none';renderCo();coT=GT;}}
function updateBar(){
 bar.innerHTML='';
 const add=(tx,fn,cls)=>{const b=document.createElement('button');b.textContent=tx;if(cls)b.className=cls;b.onclick=fn;bar.appendChild(b);};
 add(lang==='ar'?'EN':'عربي',toggleLang);
 add(t('shop'),openShop);
 add(t('pcBtn'),togglePC);
 add(t('coBtn'),toggleCo);
 if(placing){add(`📍 ${nm(placing)}`,()=>{},'on');add(t('stop'),()=>{placing=null;updateBar();});}
 else{const h=document.createElement('span');h.style.cssText='font-size:13px;opacity:.7;padding:8px;white-space:nowrap';
  h.textContent=t('hint');bar.appendChild(h);}
}
function showBld(id){
 const b=NB.get(id),d=diag(b);
 const st=b.type==='house'?(b.sat>.99?'🟢 '+t('on'):b.sat>0?`⚡ ${Math.round(b.sat*100)}% ${t('sun')}`:'⚫ '+t('off'))+`  😊 ${Math.round(b.happy*100)}%`
  :b.type==='factory'?(b.wood?'🪵 '+t('mine'):'❌ '+t('noWood'))
  :b.type==='rfac'?`${Math.round(b.sat*100)}%`
  :b.type==='cafe'?`${Math.round(b.sat*100)}% · ${b.cust} ${t('cust')}`
  :b.type==='battery'?`🔋 ${Math.round(b.st/(BCAP*b.lv)*100)}%`
  :b.type==='dist'?`${DIST_CAP*b.lv} MW`
  :'🟢';
 const btns=[];
 if(b.lv<MAXLV&&TY[b.type].u>0){const c=TY[b.type].u*b.lv;btns.push([`${t('up')} Lv${b.lv+1} (${c} 💰)`,()=>upg(id),money<c]);}
 if(b.type!=='house'&&b.type!=='rfac')btns.push([`${t('del')} (+${Math.round(b.paid*.8)} 💰)`,()=>removeB(id)]);
 const staff=hasStaff(b)?`\n👷 ${EMP} ${t('emp')} — ${working(b)?t('work'):t('rest')}`:'';
 showP(`${TY[b.type].i} ${nm(b.type)} Lv${b.lv} — ${st}${staff}\n${d?t(d):t('ok')}`,btns);}
function nmE(id){return typeof id==='string'?t('forest'):nm(NB.get(id).type);}
function renderPC(){
 const sc=pc.scrollTop;
 pc.innerHTML='';
 const head=document.createElement('div');head.innerHTML=`<b>${t('pcT')}</b> `;
 const x=document.createElement('button');x.textContent='✖';x.onclick=()=>{pcOpen=false;pc.style.display='none';};head.appendChild(x);pc.appendChild(head);
 const row=(txt,cls)=>{const r=document.createElement('div');r.className='r';const s=document.createElement('div');s.textContent=txt;if(cls)s.className=cls;r.appendChild(s);pc.appendChild(r);};
 const sec=title=>{const h=document.createElement('div');h.style.cssText='margin-top:12px;font-weight:bold';h.textContent=title;pc.appendChild(h);};
 sec('🔌 '+t('cab'));
 row(t('cabNote'));
 for(const e of E){const days=Math.max(0,Math.ceil((LIFE-(GT-e.born))/DAY));
  const name=`${nmE(e.a)} ↔ ${nmE(e.b)}`;
  if(e.broken)row(`${name}: ${t('d_brk')}`,'m');
  else row(`${name}: ${days} ${t('days')}`);}
 sec('🏗️');
 for(const b of NB.values()){const d=diag(b);row(`${TY[b.type].i} ${nm(b.type)} Lv${b.lv}: ${d?t(d):t('ok')}`,d?'m':'');}
 pc.scrollTop=sc;
}

/* ---------- company dashboard ---------- */
function drawChart(){
 const c=$('chart');if(!c)return;const g=c.getContext('2d'),w=c.width,h=c.height;
 g.clearRect(0,0,w,h);
 const a=priceHist,mn=Math.min(...a),mx=Math.max(...a),rg=Math.max(1,mx-mn);
 g.strokeStyle='#223';g.lineWidth=1;
 for(let i=1;i<4;i++){g.beginPath();g.moveTo(0,h*i/4);g.lineTo(w,h*i/4);g.stroke();}
 const up=a[a.length-1]>=a[0],col=up?'#7fd69a':'#ff6b6b';
 const px=i=>i/Math.max(1,a.length-1)*(w-8)+4,py=v=>h-8-((v-mn)/rg)*(h-18);
 g.beginPath();a.forEach((v,i)=>i?g.lineTo(px(i),py(v)):g.moveTo(px(i),py(v)));
 g.strokeStyle=col;g.lineWidth=2.5;g.stroke();
 g.lineTo(px(a.length-1),h);g.lineTo(px(0),h);g.closePath();g.globalAlpha=.12;g.fillStyle=col;g.fill();g.globalAlpha=1;
 g.fillStyle='#9ab';g.font='11px sans-serif';g.textAlign='left';g.fillText(mx.toFixed(1),4,11);g.fillText(mn.toFixed(1),4,h-2);}
function renderCo(){
 const sc=co.scrollTop;
 const last=priceHist[priceHist.length-1],prev=priceHist.length>1?priceHist[priceHist.length-2]:last,first=priceHist[0];
 const chg=(last-prev)/prev*100,tot=(last-first)/first*100,up=last>=prev;
 let sts=0,work=0,hs=0,sat=0,hap=0,blds=0;
 for(const b of NB.values()){
  if(hasStaff(b)){sts++;if(working(b))work++;}
  if(!b.auto&&b.type!=='house')blds++;
  if(b.type==='house'){hs++;sat+=b.sat;hap+=b.happy;}}
 const margin=rates.rev>.01?rates.net/rates.rev*100:0;
 const totNet=stat.rev-stat.exp;
 const f=n=>Math.round(n).toLocaleString('en');
 const row=(a,b,cls)=>`<div class="kv"><span>${a}</span><b class="${cls||''}">${b}</b></div>`;
 const sg=n=>(n>=0?'+':'')+f(n);
 co.innerHTML=
 `<div><b>${t('coT')}</b> <button id="coX">✖</button></div>
 <div style="opacity:.7;font-size:12px;margin-top:6px">${t('share')}</div>
 <div class="px ${up?'up':'dn'}">${last.toFixed(2)} ${up?'▲':'▼'} <small>${chg>=0?'+':''}${chg.toFixed(2)}%</small></div>
 <canvas id="chart" width="340" height="130"></canvas>
 ${row(t('since'),(tot>=0?'+':'')+tot.toFixed(1)+'%',tot>=0?'up':'dn')}
 ${row(t('val'),f(last*1000)+' 💰')}
 <h4>${t('perf')}</h4>
 ${row(t('rev'),f(rates.rev)+' 💰','up')}
 ${row(t('exp'),f(rates.exp)+' 💰','dn')}
 ${row(t('net')+' ('+(rates.net>=0?t('profit'):t('loss'))+')',sg(rates.net)+' 💰',rates.net>=0?'up':'dn')}
 ${row(t('margin'),margin.toFixed(1)+'%',margin>=0?'up':'dn')}
 <h4>${t('costs')}</h4>
 ${row(t('wages'),f(stat.wages)+' 💰')}
 ${row(t('upk'),f(stat.upk)+' 💰')}
 ${row(t('cabs'),f(stat.cab)+' 💰')}
 ${row(t('cafeNet'),sg(stat.cafe)+' 💰',stat.cafe>=0?'up':'dn')}
 ${row(t('lost'),f(stat.lost)+' 💰','dn')}
 <h4>${t('totNet')}</h4>
 ${row(t('totRev'),f(stat.rev)+' 💰')}
 ${row(t('totExp'),f(stat.exp)+' 💰')}
 ${row(t('totNet'),sg(totNet)+' 💰',totNet>=0?'up':'dn')}
 <h4>${t('staff')}</h4>
 ${row(t('staff'),sts*EMP+'  ('+work*EMP+' '+t('staffWork')+')')}
 ${row(t('blds'),blds)}
 ${row(t('houses'),hs)}
 ${row(t('avgSat'),hs?Math.round(sat/hs*100)+'%':'-')}
 ${row(t('happy'),hs?Math.round(hap/hs*100)+'%':'-')}
 ${row(t('evaded'),stat.evaded,'dn')}
 ${row(t('cities'),openedCount)}`;
 $('coX').onclick=()=>{coOpen=false;co.style.display='none';};
 drawChart();co.scrollTop=sc;
}

/* ---------- hit testing ---------- */
function distSeg(px,py,x1,y1,x2,y2){const dx=x2-x1,dy=y2-y1;const l=dx*dx+dy*dy;
 let k=l?((px-x1)*dx+(py-y1)*dy)/l:0;k=Math.max(0,Math.min(1,k));return Math.hypot(px-(x1+dx*k),py-(y1+dy*k));}
function hitAt(px,py){
 for(const b of NB.values()){const [sx,sy]=toS(b.x,b.y);if(Math.hypot(px-sx,py-sy)<20)return {kind:'b',id:b.id};}
 for(const st of stores){const [sx,sy]=toS(st.x,st.y);if(Math.hypot(px-sx,py-sy)<20)return {kind:'store'};}
 const [wx,wy]=toW(px,py);
 for(const f of forestsRect(wx-120,wy-120,wx+120,wy+120))if(Math.hypot(wx-f.x,wy-f.y)<f.r)return {kind:'forest',f};
 for(const e of E){const A=node(e.a),B=node(e.b);if(!A||!B)continue;
  const [p,q]=toS(A.x,A.y),[r,tt2]=toS(B.x,B.y);if(distSeg(px,py,p,q,r,tt2)<8)return {kind:'cable',e};}
 for(const c of cities)if(!c.open&&Math.abs(wx-c.x)<HALF&&Math.abs(wy-c.y)<HALF)return {kind:'city',c};
 return null;}

/* ---------- input ---------- */
const P=new Map();let drag=null,pinch=null;
function place(px,py){
 const [x,y]=toW(px,py);const err=canPlace(placing,x,y);
 if(err){toast(err);return;}
 const c=costAt(placing,x,y);
 if(money<c){toast(t('noM'));return;}
 money-=c;
 if(placing==='bridge')bridges.push({x,y});else addB(placing,x,y,c);}
cv.addEventListener('pointerdown',e=>{
 P.set(e.pointerId,{x:e.clientX,y:e.clientY});mouse={x:e.clientX,y:e.clientY};
 if(P.size===2){const [a,b]=[...P.values()];pinch={d:Math.hypot(a.x-b.x,a.y-b.y)};drag=null;return;}
 const h=hitAt(e.clientX,e.clientY);
 drag={sx:e.clientX,sy:e.clientY,x:e.clientX,y:e.clientY,px:e.clientX,py:e.clientY,moved:false,hit:h,link:null};
 if(h&&h.kind==='b')drag.link=h.id;
 else if(h&&h.kind==='forest')drag.link=h.f.id;
});
cv.addEventListener('pointermove',e=>{
 mouse={x:e.clientX,y:e.clientY};
 if(!P.has(e.pointerId))return;
 P.get(e.pointerId).x=e.clientX;P.get(e.pointerId).y=e.clientY;
 if(pinch&&P.size===2){const [a,b]=[...P.values()];const d=Math.hypot(a.x-b.x,a.y-b.y);
  zoomAt((a.x+b.x)/2,(a.y+b.y)/2,d/pinch.d);pinch.d=d;return;}
 if(!drag)return;
 const dx=e.clientX-drag.x,dy=e.clientY-drag.y;drag.x=e.clientX;drag.y=e.clientY;
 drag.px=e.clientX;drag.py=e.clientY;
 if(Math.hypot(e.clientX-drag.sx,e.clientY-drag.sy)>6)drag.moved=true;
 if(!drag.link&&drag.moved){cam.x-=dx/cam.s;cam.y-=dy/cam.s;}
});
function endPointer(e){
 const had=P.has(e.pointerId);P.delete(e.pointerId);
 if(pinch){if(P.size<2)pinch=null;drag=null;return;}
 if(!drag||!had){drag=null;return;}
 const d=drag;drag=null;
 if(d.link&&d.moved){
  const h=hitAt(e.clientX,e.clientY);
  const tid=h&&h.kind==='b'?h.id:h&&h.kind==='forest'?h.f.id:null;
  if(tid!==null&&tid!==d.link)connect(d.link,tid);
  return;}
 if(d.moved)return;
 if(placing){place(e.clientX,e.clientY);return;}
 const h=d.hit;
 if(!h){closePanel();return;}
 if(h.kind==='b')showBld(h.id);
 else if(h.kind==='store')openShop();
 else if(h.kind==='forest')showP('🌲 '+t('frInfo'));
 else if(h.kind==='cable'){const e2=h.e;
  const days=Math.max(0,Math.ceil((LIFE-(GT-e2.born))/DAY));
  const del=[t('cabDel'),()=>{const i=E.indexOf(e2);if(i>=0)E.splice(i,1);closePanel();}];
  if(e2.broken)showP(`🔌 ${t('cab')} — ${t('brk')}\n${t('d_brk')}`,[del]);
  else showP(`🔌 ${t('cab')} — ${days} ${t('days')}\n${t('cabNote')}`,[del]);}
 else if(h.kind==='city'){const c=h.c,cost=nextCost(openedCount-1);
  showP(t('locked'),[[`${t('open')} — ${cost} 💰`,()=>openC(c),money<cost]]);}
}
function openC(c){const cost=nextCost(openedCount-1);
 if(money<cost){toast(t('noM'));return;}
 money-=cost;c.open=true;openedCount++;buildCityContents(c);spawnNear(c);closePanel();toast(t('opened'));}
cv.addEventListener('pointerup',endPointer);
cv.addEventListener('pointercancel',endPointer);
cv.addEventListener('wheel',e=>{e.preventDefault();zoomAt(e.clientX,e.clientY,e.deltaY<0?1.1:1/1.1);},{passive:false});

/* ---------- HUD & loop ---------- */
function hud(){
 const m=$('money');m.textContent='💰 '+Math.floor(money);
 m.style.color=money>lastMoney+.5?'#7fd69a':money<lastMoney-.5?'#ff8a8a':'#e8efe9';lastMoney=money;
 const n=Math.round(rates.net);
 $('inc').textContent=`📈 ${n>=0?'+':''}${n} ${t('perMin')}  ${priceHist[priceHist.length-1]>=(priceHist[priceHist.length-2]||0)?'▲':'▼'}${price.toFixed(0)}`;
 const rem=gDay?30-(GT%DAY):DAY-(GT%DAY);
 $('time').textContent=(gDay?'☀️ ':'🌙 ')+(gDay?t('day'):t('night'))+' '+Math.ceil(rem)+'s';}
updateBar();
let last=performance.now();
function frame(now){
 const dt=Math.min(.1,(now-last)/1000);last=now;lastDt=dt;
 GT+=dt;gDay=(GT%DAY)<DAY/2;
 sim(dt);
 statT+=dt;if(statT>=5){statT-=5;sample();}
 // residents leave their houses for the cafés nearby
 walkT+=dt;if(walkT>.8){walkT=0;
  for(const c of NB.values())if(c.type==='cafe'&&c.sat>0&&walkers.length<60){
   const hs=[...NB.values()].filter(h=>h.type==='house'&&h.sat>.3&&Math.hypot(h.x-c.x,h.y-c.y)<CAFE_R);
   if(hs.length)walkers.push({a:hs[Math.floor(Math.random()*hs.length)],b:c,t:0});}}
 // safety net: no money and no income means the city grants an emergency sum, so the game never gets stuck
 if(money<1&&rates.rev<1&&GT-lastGrant>120){money+=200;lastGrant=GT;toast(t('grant'));}
 drawAll();hud();
 if(pcOpen&&GT-pcT>1){renderPC();pcT=GT;}
 if(coOpen&&GT-coT>1){renderCo();coT=GT;}
 requestAnimationFrame(frame);}
requestAnimationFrame(frame);
</script>
</body>
</html>
