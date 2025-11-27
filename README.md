<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <title>Universal Bio AI Annotator</title>
  <style>
    body{font-family:Arial,Helvetica,sans-serif;margin:0;padding:20px;background:#f9f9f9}
    h1{margin-top:0}
    canvas{border:1px solid #ccc;cursor:crosshair;background:#fff;display:block;margin-bottom:10px}
    button{padding:6px 12px;margin-right:6px}
    select{padding:4px}
  </style>
</head>
<body>
  <h1>🔬 Universal Bio AI Annotator</h1>
  <input type="file" id="imgIn" accept="image/*"/>
  <br/><br/>
  <canvas id="c" width="512" height="512"></canvas>
  <br/>
  <label>Class:</label>
  <select id="cls"></select>
  <button id="saveBtn">💾 Download Mask</button>

  <script>
  /* ===== CONFIG ===== */
  const CFG={
    classes:{
      0:{name:"background",color:[0,0,0]},
      1:{name:"nucleus",   color:[255,0,0]},
      2:{name:"membrane",  color:[0,255,0]},
      3:{name:"cytoplasm", color:[0,0,255]}
    }
  };

  /* ===== INIT ===== */
  const c=document.getElementById('c'), ctx=c.getContext('2d');
  const clsSel=document.getElementById('cls');
  Object.entries(CFG.classes).forEach(([id,{name,color}])=>{
    const o=document.createElement('option'); o.value=id; o.text=`${name} (RGB:${color})`; clsSel.appendChild(o);
  });
  let curr=0, drawing=false, img=new Image();

  /* ===== LOAD IMAGE ===== */
  document.getElementById('imgIn').onchange=e=>{
    const fr=new FileReader();
    fr.onload=()=>{img.onload=()=>{ctx.drawImage(img,0,0,c.width,c.height);}; img.src=fr.result;};
    fr.readAsDataURL(e.target.files[0]);
  };

  /* ===== DRAW ===== */
  clsSel.onchange=e=>curr=parseInt(e.target.value);
  c.onmousedown=()=>drawing=true;
  c.onmouseup=()=>drawing=false;
  c.onmousemove=e=>{
    if(!drawing)return;
    const rect=c.getBoundingClientRect(), x=e.clientX-rect.left, y=e.clientY-rect.top;
    const [r,g,b]=CFG.classes[curr].color;
    ctx.fillStyle=`rgb(${r},${g},${b})`;
    ctx.beginPath(); ctx.arc(x,y,4,0,Math.PI*2); ctx.fill();
  };

  /* ===== EXPORT MASK ===== */
  document.getElementById('saveBtn').onclick=()=>{
    const idt=ctx.getImageData(0,0,c.width,c.height), d=idt.data;
    const out=new Uint8ClampedArray(d.length);
    for(let i=0;i<d.length;i+=4){
      const r=d[i], g=d[i+1], b=d[i+2];
      let set=[0,0,0];
      for(const [cid,{color:[cr,cg,cb]}] of Object.entries(CFG.classes)){
        if(r===cr&&g===cg&&b===cb){set=[cr,cg,cb];break;}
      }
      out[i]=set[0]; out[i+1]=set[1]; out[i+2]=set[2]; out[i+3]=255;
    }
    const oc=document.createElement('canvas'); oc.width=c.width; oc.height=c.height;
    oc.getContext('2d').putImageData(new ImageData(out,c.width,c.height),0,0);
    oc.toBlob(b=>{
      const a=document.createElement('a'), u=URL.createObjectURL(b);
      a.href=u; a.download='mask.png'; a.click(); URL.revokeObjectURL(u);
    });
  };
  </script>
</body>
</html>
