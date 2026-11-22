<div align="center">

# 🔥 EMBER HD

### `spark ➔ candle ➔ fall ➔ inferno`
**282×88 ASCII Cells &nbsp;•&nbsp; 16:9 Cellular Automata Simulation**

<br>

<!-- ═══════════════════════ EMBER HD ANIMATED CELLULAR SIMULATION ═══════════════════════ -->
<a href="https://fayaspsulfikkar.github.io/fayaspsulfikkar/">
  <img src="assets/ember_inferno.gif" alt="EMBER HD ASCII Cellular Inferno Animation" width="100%" style="max-width: 720px; border-radius: 8px;" />
</a>

<br><br>

[![Launch Interactive Simulation](https://img.shields.io/badge/▶_Launch_Live_Interactive_Simulation-FF5500?style=for-the-badge&logo=fire&logoColor=white)](https://fayaspsulfikkar.github.io/fayaspsulfikkar/)
&nbsp;&nbsp;
[![Reset Sequence](https://img.shields.io/badge/⌨_Press_'R'_to_Reset-161b22?style=for-the-badge&logo=github&logoColor=white)](https://fayaspsulfikkar.github.io/fayaspsulfikkar/)

<p>
  <sub>Click the simulation above or launch the live web app to run real-time cellular combustion in your browser.</sub>
</p>

</div>

<br>

<details open>
<summary><b>📄 Simulation Source Code (JavaScript)</b></summary>

```javascript
/* ===== EMBER HD (room from your photo): spark -> candle -> fall -> inferno, in 282x88 ASCII cells (16:9) =====
   Units (U) = the old 200x84 room. The simulation runs on a grid S times finer.
   Phases are driven by one clock t. Fire = real cellular simulation. Glyphs are blitted as pixels. */
const S=2,UW=200,UH=84,W=UW*S,H=UH*S,N=W*H,R=88,G=66,AS=1.8,CY=46,CAND=80,PV=CAND+7,PW=720,PHh=405,PHU=62.53,PY0=CY-PHU/2;
const C=Math.round(R*AS*16/9);                       // columns for a 16:9 picture
const T_SPARK=1.2,T_HIT=4.6,T_ZOOM=8.8,T_FALL=11.9,T_END=54;
const cv=document.getElementById('c'),cx=cv.getContext('2d'),bl=document.getElementById('bl'),bx=bl.getContext('2d');
const rnd=Math.random,clamp=(v,a,b)=>v<a?a:v>b?b:v,sm=p=>{p=clamp(p,0,1);return p*p*(3-2*p)};
const hs=(a,b)=>{const n=Math.sin(a*127.1+b*311.7)*43758.5453;return n-Math.floor(n)};
const nz=(x,y)=>hs(x*5.3,y*3.7)-.5;
let cw,chh,zEnd,chs=new Array(C*R).fill(''),cls=new Int32Array(C*R),img,u32,mk,mg;const masks=new Map();

/* ---------- colour ramps (packed r|g<<8|b<<16) ---------- */
const pk=(r,g,b)=>r|g<<8|b<<16,WL=[],FL_=[],BLUE=pk(110,150,255);
for(let k=0;k<64;k++){const l=k/63;
 WL.push(pk(255*Math.pow(Math.min(1,l*1.1),.8)|0,190*Math.pow(l,1.25)|0,90*Math.pow(l,2.2)|0));
 FL_.push(pk(255*Math.min(1,l*1.9)|0,255*clamp((l-.3)*1.8,0,1)|0,255*clamp((l-.82)*5,0,1)|0));}
const RAMP=" .`'^\",:;Il!i><~+_-?][}{1)(|/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$",RL=RAMP.length,FCH=".:,;+*x%#&@";

/* ---------- the room = your photo. Brightness comes from the image; materials from zones ---------- */
const T=new Uint8Array(N),PHa=new Uint8Array(PW*PHh); // T: 0 never burns,1 wood,2 paper,3 leather/cloth,4 panel
const MR=[0,.0035,.011,.015,.003],MI=[0,.12,.4,.55,.12];
const PR=[722/1672*PW,1122/1672*PW,94/941*PHh,596/941*PHh]; // portrait + frame (kept intact)
function zone(x,y){ // x,y in original photo pixels (1672x941)
 if(y<0)return 4;if(y>941)return 1;
 if(x>=722&&x<=1122&&y>=94&&y<=596)return 0;          // portrait and its frame
 if(x>=805&&x<=1030&&y>=38&&y<=66)return 0;           // picture light
 if(x>=425&&x<=602&&y>=543&&y<=775)return 0;          // brass lamp
 if(x>=795&&x<=1040&&y>=596&&y<752)return 3;          // leather chair
 if(x>=1195&&x<=1358&&y>=685&&y<=772)return 2;        // books and pen cup
 if(x>=705&&x<=1160&&y>=748&&y<=782)return 2;         // desk pad and laptop
 if(y>=752&&x>=275&&x<=1540)return 1;                 // desk
 if(x>=1545)return 1;                                 // shutters
 return 4}                                            // panelled walls and floor
function build(){for(let y=0;y<H;y++){const qy=((y/S-PY0)/PHU)*941;for(let x=0;x<W;x++)T[y*W+x]=zone(x/S/UW*1672,qy)}}
function loadPhoto(cb){const im=new Image();im.onload=()=>{const c=document.createElement('canvas');c.width=PW;c.height=PHh;const g=c.getContext('2d');
 g.drawImage(im,0,0,PW,PHh);const d=g.getImageData(0,0,PW,PHh).data;for(let i=0;i<PW*PHh;i++)PHa[i]=d[i*4];cb()};
 im.src=document.getElementById('photo').textContent.trim()}
function ph(x,y){x=clamp(x,0,PW-1.001);y=clamp(y,0,PHh-1.001);const xi=x|0,yi=y|0,fx=x-xi,fy=y-yi,i=yi*PW+xi;
 return((PHa[i]*(1-fx)+PHa[i+1]*fx)*(1-fy)+(PHa[i+PW]*(1-fx)+PHa[i+PW+1]*fx)*fy)/255}

/* ---------- state ---------- */
const F=new Float32Array(N),fuel=new Float32Array(N),coal=new Float32Array(N),burn=new Uint8Array(N);
const GW=34,GH=14,gl=new Float32Array(GW*GH),g2=new Float32Array(GW*GH);
let P=[],trail=[],glow=0,simT=0,t0=0,flamTotal=0,burst=false,flashed=false;
function reset(){F.fill(0);coal.fill(0);burn.fill(0);flamTotal=0;
 for(let i=0;i<N;i++){fuel[i]=T[i]?1:0;if(T[i])flamTotal++}
 P=[];trail=[];glow=0;simT=0;burst=false;flashed=false;t0=performance.now()}

/* ---------- choreography: a function of time ---------- */
function theta(t){const s=t-T_FALL;if(s<=0)return 0;const tl=T_FALL+.91;
 if(t<tl)return Math.min(1.5708,1.9*s*s+.05*s);const q=t-tl;return 1.5708-.07*Math.exp(-6*q)*Math.abs(Math.sin(17*q))}
function wick(t){const th=theta(t),c=Math.cos(th),s=Math.sin(th);return[PV+(-7*c+44*s),G-(7*s+44*c)/AS]}
function spark(t){
 if(t<T_SPARK||t>=T_HIT)return null;const p=(t-T_SPARK)/(T_HIT-T_SPARK),u=1-Math.pow(1-p,2.2),v=1-u,[ex,ey]=wick(0);
 const x=v*v*v*14+3*v*v*u*38+3*v*u*u*60+u*u*u*ex,y=v*v*v*22+3*v*v*u*64+3*v*u*u*18+u*u*u*ey,w=Math.sin(t*15)*1.6*v;
 return[x+w*.4,y+w]}
const flameSize=t=>{const s=t-T_HIT;return s<0?0:sm(s/1.5)+.25*Math.exp(-((s-.45)**2)/.05)};
const zoom=t=>1+(zEnd-1)*sm((t-T_ZOOM)/2.6);

/* ---------- fire simulation (30 Hz, fine grid) ---------- */
function put(i,v){if(v>F[i])F[i]=v}
function step(ts){
 const [fx,fy]=wick(ts),th=theta(ts),sp=spark(ts);
 if(sp){trail.push([sp[0],sp[1]]);if(trail.length>24)trail.shift();
  P.push({x:sp[0],y:sp[1],vx:(rnd()-.5)*5,vy:rnd()*4,l:.5+rnd()*.6,m:1,k:0})}
 else if(trail.length)trail.shift();
 if(ts>=T_HIT&&!burst){burst=true;for(let k=0;k<60;k++){const a=rnd()*6.28,s=6+rnd()*18;
  P.push({x:fx,y:fy,vx:Math.cos(a)*s,vy:Math.sin(a)*s*.6-6,l:.5+rnd()*.9,m:1,k:0})}}
 if(ts>T_FALL+.4){const e=Math.min(1.15,(ts-T_FALL-.4)*1.3),X=(fx*S)|0,Y=(fy*S)|0; // the fallen flame feeds the fire
  for(let dy=-6;dy<=2;dy++)for(let dx=-4;dx<=4;dx++){const x=X+dx,y=Y+dy;if(x>=0&&x<W&&y>=0&&y<H)put(y*W+x,e*(1-.1*Math.abs(dx)))}}
 if(Math.abs(th-1.5708)<.12&&!flashed){flashed=true;for(let k=0;k<60;k++){const a=-rnd()*3.1,s=10+rnd()*22;
  P.push({x:fx,y:fy,vx:Math.cos(a)*s,vy:Math.sin(a)*s,l:.8+rnd(),m:1,k:1})}}
 for(let x=0;x<W;x++)F[(H-1)*W+x]*=.8;
 for(let y=0;y<H-1;y++)for(let x=0;x<W;x++){ // heat rises, cools, wobbles sideways
  const sx=clamp(x+(rnd()*3|0)-1,0,W-1),v=F[(y+1)*W+sx]-rnd()*.043;F[y*W+x]=v>0?v:0}
 let bc=0;const acc=1+6*glow;
 for(let i=0;i<N;i++){const m=T[i];
  if(m&&fuel[i]>0){
   if(!burn[i]){const h=F[i];if(h>.22&&rnd()<1.5*MI[m]*acc*Math.min(1,h*1.5))burn[i]=1}
   else{bc++;fuel[i]-=MR[m]*(.7+rnd()*.6);
    if(fuel[i]<=0){fuel[i]=0;burn[i]=0;coal[i]=1}
    else{const s=(.55+.5*fuel[i])*(.7+.3*rnd());put(i,s);if(rnd()<.5){const x=i%W,j=i+(rnd()<.5?-1:1);if(x>0&&x<W-1)put(j,s*.6)}}}}
  if(coal[i]>0){coal[i]-=.0007;put(i,coal[i]*.38*(.6+.8*rnd()))}}
 glow+=(Math.min(1,bc/(flamTotal*.45))-glow)*.04;
 for(let k=0;k<5;k++){const i=rnd()*N|0;if(burn[i]&&rnd()<.5)P.push({x:(i%W)/S,y:((i/W)|0)/S,vx:(rnd()-.5)*16,vy:-(8+rnd()*26),l:1+rnd()*1.5,m:1,k:1})}
 gl.fill(0);for(let i=0;i<N;i++)if(F[i]>.05)gl[(((i/W)|0)/12|0)*GW+((i%W)/12|0)]+=F[i]/144;
 for(let y=0;y<GH;y++)for(let x=0;x<GW;x++){let s=0,n=0;for(let a=-1;a<=1;a++)for(let b=-1;b<=1;b++){const X=x+b,Y=y+a;if(X>=0&&X<GW&&Y>=0&&Y<GH){s+=gl[Y*GW+X];n++}}g2[y*GW+x]=s/n}
 for(let k=P.length-1;k>=0;k--){const p=P[k];p.x+=p.vx/30;p.y+=p.vy/30;p.vy+=(p.k?30:10)/30;p.l-=1/30;
  const i=((p.y*S)|0)*W+((p.x*S)|0);
  if(p.k&&p.vy>0&&p.x>=0&&p.x<UW&&p.y>=0&&p.y<UH&&T[i]&&fuel[i]>.05){put(i,.9);if(rnd()<.3)p.l=0}
  if(p.l<=0||p.y>UH||p.x<0||p.x>UW)P.splice(k,1)}
}

/* ---------- drawing: pick a glyph + colour per cell, then blit glyph pixels ---------- */
function mask(ch){let m=masks.get(ch);if(!m){mg.clearRect(0,0,cw,chh);mg.fillStyle='#fff';mg.fillText(ch,cw/2,chh/2);
 const d=mg.getImageData(0,0,cw,chh).data;m=new Uint8Array(cw*chh);for(let i=0;i<m.length;i++)m[i]=d[i*4+3];masks.set(ch,m)}return m}
function draw(t){
 const z=zoom(t)*S,sh=(Math.max(0,1.5*Math.exp(-(t-T_FALL-.91)*2.5))*(t>T_FALL+.9?1:0)+glow*.5+(t>T_FALL?.12:0))*(rnd()-.5)*2,
  shx=sh,shy=sh*.6,th=theta(t),ct=Math.cos(th),st=Math.sin(th),[fx,fy]=wick(t),f=flameSize(t),sp=spark(t),
  FI=f*(.85+.15*Math.sin(t*17)+.08*Math.sin(t*31+1))*(t<T_HIT+.6?1+1.2*Math.exp(-(t-T_HIT)*2.5):1),
  amb=.9*glow,lcx=fx,lcy=fy-6,cX=CAND+(100-CAND)*sm((t-T_ZOOM)/2.6),rv=.2+.8*sm((t-T_FALL-1.5)/8);
 chs.fill('');
 for(let sy=0;sy<R;sy++)for(let sx=0;sx<C;sx++){
  const wx=cX+(sx+.5-C/2)/z+shx,wy=CY+(sy+.5-R/2)/z+shy,xi=Math.floor(wx*S),yi=Math.floor(wy*S);
  if(xi<0||xi>=W||yi<0||yi>=H)continue;
  const i=yi*W+xi,o=sy*C+sx;
  const qx=wx/UW*PW,qy=(wy-PY0)/PHU*PHh,ex=.25*PW/UW/z,ey=.25*PHh/PHU/z,inP=qx>=PR[0]&&qx<=PR[1]&&qy>=PR[2]&&qy<=PR[3];
  let a=(ph(qx-ex,qy-ey)+ph(qx+ex,qy-ey)+ph(qx-ex,qy+ey)+ph(qx+ex,qy+ey))*.25*rv*(inP?1.3:1)*(T[i]?1-.6*(1-fuel[i]):1),ch='',col=0;
  const px=wx-PV,py=(G-wy)*AS,uu=px*ct-py*st+7,v=px*st+py*ct; // candle space: uu across, v up
  if(Math.abs(uu)<=7&&v>=0&&v<=39.6){const cy=Math.sqrt(1-(uu/7.3)**2);
   a=.4+.5*cy+.05*hs(Math.floor(uu*3),Math.floor(v*.4));if(v>38)a=.95;else if(v>39.6-7*hs(Math.floor(uu*.8),9)&&Math.abs(uu)>2)a+=.18}
  else if(Math.abs(uu)<.8&&v>39.6&&v<45)a=.15;
  const dx=wx-lcx,dy=(wy-lcy)*AS;let L=FI*300/(dx*dx+dy*dy+80)+amb+1.7*g2[((yi/12)|0)*GW+((xi/12)|0)];
  if(sp){const a2=wx-sp[0],b2=(wy-sp[1])*AS;L+=110/(a2*a2+b2*b2+40)}
  const l=Math.min(1,a*L);
  if(l>=.035){ch=RAMP[1+Math.min(RL-2,(Math.pow(l,.75)*(RL-1))|0)];col=WL[(l*63)|0]}
  const c=coal[i];if(c>.04&&fuel[i]<=0){const q=c*.5*(.7+.3*Math.sin(t*3+xi*.45+yi*.5));if(q>l){ch=q>.25?':':'.';col=FL_[(q*63)|0]}}
  const h=F[i];if(h>(inP?.62:.28)){ch=FCH[Math.min(10,(h*10+rnd()*2)|0)];col=FL_[Math.min(63,(h*63)|0)]}
  if(sp){const ds=Math.hypot(wx-sp[0],(wy-sp[1])*AS),q=1-ds/5.5;if(q>.12){ch=q>.8?'*':q>.45?'+':':';col=FL_[(Math.min(1,q+.15)*63)|0]}}
  if(f>.02&&Math.abs(wx-fx)<9){const s=((fy-wy)*AS)/(25*f);
   if(s>-.02&&s<1){const sw=Math.sin(t*9+s*6)*1.4*s+Math.sin(t*23+s*11)*.5*s,
     w=4.2*f*Math.sin(Math.PI*Math.pow(Math.max(s,0),.55))*(1+.1*Math.sin(t*13+s*9)),d=Math.abs(wx-fx-sw);
    if(d<w){const hot=1-d/w*.6-s*.5;ch=FCH[clamp((hot*10+rnd()*1.5)|0,0,10)];col=FL_[clamp((hot*1.15*63)|0,0,63)];
     if(s<.14&&d<w*.55){ch=':';col=BLUE}}}}
  if(ch){chs[o]=ch;cls[o]=col}}
 const mark=(wx,wy,ch,col)=>{const sx=Math.floor((wx-cX-shx)*z+C/2),sy=Math.floor((wy-CY-shy)*z+R/2);if(sx>=0&&sx<C&&sy>=0&&sy<R){chs[sy*C+sx]=ch;cls[sy*C+sx]=col}};
 trail.forEach((p,k)=>mark(p[0],p[1],k>20?'*':k>12?'+':'.',FL_[(8+k*2)|0]));
 P.forEach(p=>{const q=clamp(p.l/p.m,0,1);mark(p.x,p.y,q>.5?'*':'.',FL_[(20+q*40)|0])});
 const fade=1-clamp((t-T_END)/3,0,1),Wp=C*cw;u32.fill(0xff000000);
 for(let o=0;o<C*R;o++){const ch=chs[o];if(!ch)continue;const m=mask(ch),col=cls[o],
   r=((col&255)*fade)|0,g=(((col>>8)&255)*fade)|0,b=(((col>>16)&255)*fade)|0,x0=(o%C)*cw,y0=((o/C)|0)*chh;
  for(let y=0;y<chh;y++){let p=(y0+y)*Wp+x0,q=y*cw;for(let x=0;x<cw;x++,p++,q++){const k=m[q];
   if(k){const e=k+1;u32[p]=0xff000000|((b*e>>8)<<16)|((g*e>>8)<<8)|(r*e>>8)}}}}
 cx.putImageData(img,0,0);bx.clearRect(0,0,bl.width,bl.height);bx.drawImage(cv,0,0,bl.width,bl.height);
}

/* ---------- layout + loop ---------- */
function fit(){const dpr=devicePixelRatio||1,aw=innerWidth*dpr,ah=(innerHeight-52)*dpr;
 cw=clamp(Math.floor(Math.min(aw/C,ah/(R*AS))),3,8);chh=Math.round(cw*AS);   // glyph cell in pixels
 cv.width=C*cw;cv.height=R*chh;bl.width=cv.width>>2;bl.height=cv.height>>2;
 const ar=cv.width/cv.height,w=Math.min(innerWidth,(innerHeight-52)*ar);
 cv.style.width=w+'px';cv.style.height=w/ar+'px';
 img=cx.createImageData(cv.width,cv.height);u32=new Uint32Array(img.data.buffer);
 mk=document.createElement('canvas');mk.width=cw;mk.height=chh;mg=mk.getContext('2d',{willReadFrequently:true});
 mg.font=`${Math.min(cw/.6,chh*.9)}px ui-monospace,Menlo,Consolas,monospace`;mg.textAlign='center';mg.textBaseline='middle';masks.clear();
 zEnd=Math.max(.6,C/(S*UW)*1.04)}
let last=0;
function frame(now){const t=(now-t0)/1000;
 if(t-simT>.5)simT=t-.1;
 while(simT<t){step(simT);simT+=1/30}
 if(now-last>=24){last=now;draw(t)}
 if(t>T_END+3)reset();
 requestAnimationFrame(frame)}
addEventListener('resize',fit);
addEventListener('keydown',e=>{if(e.key==='r'||e.key==='R')reset()});
rp.onclick=reset;
sc.onclick=()=>{src.textContent=document.getElementById('js').textContent;code.classList.add('on')};
cl.onclick=()=>code.classList.remove('on');
cp.onclick=async()=>{try{await navigator.clipboard.writeText(src.textContent);cp.textContent='Copied'}catch(e){cp.textContent='Select and copy'}setTimeout(()=>cp.textContent='Copy',1600)};
fit();build();loadPhoto(()=>{reset();requestAnimationFrame(frame)});
```
</details>
