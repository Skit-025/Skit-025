<div align="center">
<style>
@import url('https://fonts.googleapis.com/css2?family=Syncopate:wght@400;700&family=JetBrains+Mono:wght@100;300;400;700&family=Space+Grotesk:wght@300;500;700&display=swap');

#ethereal-container {
    width: 100%;
    max-width: 950px;
    height: 650px;
    background: #000000;
    position: relative;
    overflow: hidden;
    font-family: 'Space Grotesk', sans-serif;
    border: 1px solid rgba(0, 255, 64, 0.15);
    box-shadow: 0 0 100px -20px rgba(0, 255, 64, 0.1);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    border-radius: 12px;
}

canvas#matrix-3d {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    z-index: 1;
}

/* --- TYPOGRAPHY & DATA HUD --- */
.data-layer {
    position: relative;
    z-index: 15;
    text-align: center;
    color: #fff;
    pointer-events: none;
    margin-top: 300px; /* Push titles below the centered globe */
}

.auth-badge {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 6px 12px;
    background: transparent;
    border-left: 2px solid #00ff55;
    border-right: 2px solid #00ff55;
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    font-weight: 300;
    letter-spacing: 4px;
    color: rgba(255, 255, 255, 0.6);
    margin-bottom: 12px;
    text-transform: uppercase;
}

.auth-badge span {
    display: block;
    width: 6px;
    height: 6px;
    background: #00ff55;
    box-shadow: 0 0 10px #00ff55;
    border-radius: 1px;
    animation: pulse 2s infinite;
}

.title-name {
    font-family: 'Syncopate', sans-serif;
    font-size: clamp(20px, 4vw, 36px);
    font-weight: 700;
    letter-spacing: 6px;
    margin: 0;
    color: #fff;
    text-transform: uppercase;
    text-shadow: 
        0 0 10px rgba(0, 255, 85, 0.3),
        2px 2px 0px rgba(0, 255, 85, 0.2),
        -2px -2px 0px rgba(0, 0, 0, 0.8);
}

.ethereal-type {
    font-family: 'JetBrains Mono', monospace;
    font-size: 14px;
    color: #00ff55;
    font-weight: 100;
    height: 20px;
    margin-top: 10px;
    letter-spacing: 2px;
    text-shadow: 0 0 10px rgba(0,255,85,0.4);
}

.ethereal-cursor {
    display: inline-block;
    width: 10px;
    height: 14px;
    background: #00ff55;
    vertical-align: middle;
    margin-left: 4px;
    animation: blink 1s step-end infinite;
    box-shadow: 0 0 8px #00ff55;
}

/* --- KEYFRAMES --- */
@keyframes blink { 50% { opacity: 0; } }
@keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }
</style>

<div id="ethereal-container">
    <canvas id="matrix-3d"></canvas>

    <div class="data-layer">
        <div class="auth-badge"><span></span> LEVEL 9 CLEARANCE // AI/ML CORE</div>
        <h1 class="title-name">Aditya Prasad Barik</h1>
        <div class="ethereal-type">
            <span id="target-ethereal"></span><span class="ethereal-cursor"></span>
        </div>
    </div>
</div>

<script>
// Load profile picture to draw inside the canvas
const profileImg = new Image();
profileImg.src = 'profile.jpeg'; // Load from repo

const canvas = document.getElementById('matrix-3d');
const ctx = canvas.getContext('2d');
const parent = document.getElementById('ethereal-container');

let width, height, cx, cy;
function resize() {
    width = canvas.width = parent.offsetWidth;
    height = canvas.height = parent.offsetHeight;
    cx = width / 2;
    cy = height / 2 - 50; 
}
window.addEventListener('resize', resize);
resize();

class Point3D {
    constructor(x, y, z) {
        this.origX = x; this.origY = y; this.origZ = z;
        this.x = x; this.y = y; this.z = z;
    }
    rotateX(angle) {
        let r_y = Math.cos(angle) * this.y - Math.sin(angle) * this.z;
        let r_z = Math.sin(angle) * this.y + Math.cos(angle) * this.z;
        this.y = r_y; this.z = r_z;
    }
    rotateY(angle) {
        let r_x = Math.cos(angle) * this.x + Math.sin(angle) * this.z;
        let r_z = -Math.sin(angle) * this.x + Math.cos(angle) * this.z;
        this.x = r_x; this.z = r_z;
    }
    rotateZ(angle) {
        let r_x = Math.cos(angle) * this.x - Math.sin(angle) * this.y;
        let r_y = Math.sin(angle) * this.x + Math.cos(angle) * this.y;
        this.x = r_x; this.y = r_y;
    }
    project(fov, viewDist) {
        let factor = fov / (viewDist + this.z);
        return {
            px: this.x * factor + cx,
            py: this.y * factor + cy,
            scale: factor
        };
    }
}

// Generate Stunning Travel Space Stars (1500)
const stars = [];
for(let i=0; i<1500; i++){
    stars.push({
        origX: (Math.random() - 0.5) * 4000,
        origY: (Math.random() - 0.5) * 4000,
        origZ: (Math.random() - 0.5) * 4000,
        size: Math.random() * 1.5 + 0.3
    });
}

// Generate Wireframe Dotted Globe Grid
const globeGrid = [];
const radius = 240; 
for (let lat = -Math.PI/2; lat <= Math.PI/2; lat += Math.PI/12) {
    let row = [];
    for (let lon = 0; lon <= 2*Math.PI; lon += Math.PI/12) {
        let x = radius * Math.cos(lat) * Math.cos(lon);
        let y = radius * Math.sin(lat);
        let z = radius * Math.cos(lat) * Math.sin(lon);
        row.push(new Point3D(x, y, z));
    }
    globeGrid.push(row);
}

// Rockets Orbiting
const rockets = [
    { angle: 0, dist: 310, speed: 0.015, yOff: 40, rx: 0.2, rz: 0.1 },
    { angle: Math.PI/2, dist: 380, speed: -0.02, yOff: -80, rx: -0.4, rz: 0.5 },
    { angle: Math.PI, dist: 340, speed: 0.025, yOff: -10, rx: 0.8, rz: -0.3 }
];

let globalTime = 0;

function renderSpace() {
    // Solid Black Background
    ctx.fillStyle = '#000000';
    ctx.fillRect(0, 0, width, height);

    globalTime += 0.02;
    let fov = 500;
    let viewDist = 800;

    // --- 0. BACKGROUND SPACE HYPERTRAVEL PANNING (MULTI-DIRECTIONAL) ---
    for(let i=0; i<stars.length; i++){
        let s = stars[i];
        
        // Push stars forward toward camera with a pulsing speed
        let speedZ = 3 + Math.sin(globalTime * 0.02) * 2;
        s.origZ -= speedZ;
        if(s.origZ < -2000) s.origZ += 4000; // Warp back to deepest space seamlessly
        
        let p = new Point3D(s.origX, s.origY, s.origZ);
        
        // Multi-directional Camera Banking Pan
        // Shifts direction gracefully left/right and up/down
        p.rotateX(Math.sin(globalTime * 0.01) * 0.25);
        p.rotateY(Math.cos(globalTime * 0.012) * 0.25);

        let proj = p.project(fov, viewDist);
        if(proj.scale > 0 && proj.px > -50 && proj.px < width + 50 && proj.py > -50 && proj.py < height + 50) {
            let depthAlpha = Math.max(0.05, 1 - Math.abs(p.z) / 2500); 
            ctx.fillStyle = `rgba(255, 255, 255, ${depthAlpha})`;
            ctx.beginPath();
            ctx.arc(proj.px, proj.py, s.size * proj.scale, 0, Math.PI*2);
            ctx.fill();
        }
    }


    // --- 1. SINE WAVE DIAGONAL CONTINUOUS FLOW ---
    let freq = 0.006;  // More stretched out calm wave
    let amp = 100;     // Calm amplitude
    let charSpacing = 16.5;

    let baseString = "Skit-025 // Skit Here No Fear // ";
    let textStr = baseString.repeat(50); // Huge repeating buffer
    let segmentWidth = baseString.length * charSpacing;
    
    ctx.font = "bold 15px 'Syncopate', sans-serif";
    ctx.textAlign = "center";
    ctx.textBaseline = "middle";
    
    ctx.save();
    // Center logic at middle, tilt diagonally (bottom-left to top-right)
    ctx.translate(cx, cy);
    ctx.rotate(-Math.PI / 6);

    let startPos = -1200; // Start way offscreen to the left
    
    for(let i=0; i<textStr.length; i++) {
        // Slow continuous forward flow
        let xPos = startPos + i * charSpacing + (globalTime * 20) % segmentWidth;
        
        if(xPos > -800 && xPos < 800) {
            // Flow unulation // Added Math.abs to sine phase so xPos directly defines y like water flow
            let wavePhase = globalTime * 1.5; 
            let yPos = Math.sin(xPos * freq - wavePhase) * amp; 
            let deriv = freq * amp * Math.cos(xPos * freq - wavePhase);
            
            // Transparency: Fade out as it approaches the center portrait
            let distToCenter = Math.sqrt(xPos*xPos + yPos*yPos);
            let alpha = 0.9; // Highly visible white text
            
            if(distToCenter < 240) {
                alpha = (distToCenter - 110) / 130 * 0.9;
                if(alpha < 0) alpha = 0;
            }

            ctx.fillStyle = `rgba(255, 255, 255, ${alpha})`;
            ctx.save();
            ctx.translate(xPos, yPos);
            ctx.rotate(Math.atan(deriv));
            ctx.fillText(textStr[i], 0, 0);
            ctx.restore();
        }
    }
    ctx.restore();


    // --- 2. DEPTH SORT GLOBE GRID ---
    let projGrid = [];
    for(let r=0; r<globeGrid.length; r++) {
        let pRow = [];
        for(let c=0; c<globeGrid[r].length; c++) {
            let p = new Point3D(globeGrid[r][c].origX, globeGrid[r][c].origY, globeGrid[r][c].origZ);
            p.rotateX(0.3);
            p.rotateY(globalTime * 0.3);
            pRow.push({ orig: p, proj: p.project(fov, viewDist) });
        }
        projGrid.push(pRow);
    }

    // Calculate rockets
    let rocketData = [];
    rockets.forEach(r => {
        r.angle += r.speed;
        let rp = new Point3D(
            Math.cos(r.angle) * r.dist,
            r.yOff,
            Math.sin(r.angle) * r.dist
        );
        rp.rotateX(r.rx);
        rp.rotateZ(r.rz);
        rocketData.push({ orig: rp, proj: rp.project(fov, viewDist), speed: r.speed, angle: r.angle, dist: r.dist, rx: r.rx, rz: r.rz, yOff: r.yOff });
    });

    ctx.lineWidth = 0.5;

    function drawGlobe(isFrontFlag) {
        for(let r=0; r<projGrid.length; r++) {
            for(let c=0; c<projGrid[r].length; c++) {
                let pt = projGrid[r][c];
                let isFront = pt.orig.z < 0;
                if(isFront !== isFrontFlag) continue;

                if(pt.proj.scale > 0) {
                    let alpha = isFront ? 0.35 : 0.08;
                    let dotSize = isFront ? 1.5 : 0.8;
                    
                    ctx.fillStyle = `rgba(0, 255, 85, ${alpha})`;
                    ctx.beginPath();
                    ctx.arc(pt.proj.px, pt.proj.py, dotSize * pt.proj.scale, 0, Math.PI*2);
                    ctx.fill();
                    
                    ctx.strokeStyle = `rgba(0, 255, 85, ${alpha * 0.5})`;
                    
                    if(c > 0) {
                        let prevC = projGrid[r][c-1];
                        if(prevC.proj.scale > 0 && (prevC.orig.z < 0) === isFrontFlag) {
                            ctx.beginPath();
                            ctx.moveTo(pt.proj.px, pt.proj.py);
                            ctx.lineTo(prevC.proj.px, prevC.proj.py);
                            ctx.stroke();
                        }
                    }
                    if(r > 0) {
                        let prevR = projGrid[r-1][c];
                        if(prevR.proj.scale > 0 && (prevR.orig.z < 0) === isFrontFlag) {
                            ctx.beginPath();
                            ctx.moveTo(pt.proj.px, pt.proj.py);
                            ctx.lineTo(prevR.proj.px, prevR.proj.py);
                            ctx.stroke();
                        }
                    }
                }
            }
        }
    }

    function drawRockets(isFrontFlag) {
        rocketData.forEach(r => {
            let isFront = r.orig.z < 0;
            if(isFront !== isFrontFlag) return;

            if(r.proj.scale > 0) {
                let rAlpha = isFront ? 1 : 0.2;
                let dotSize = isFront ? 3 : 1;
                
                ctx.fillStyle = `rgba(0, 255, 85, ${rAlpha})`;
                ctx.shadowBlur = isFront ? 15 : 0;
                ctx.shadowColor = '#00ff55';
                ctx.beginPath();
                ctx.arc(r.proj.px, r.proj.py, dotSize * r.proj.scale, 0, Math.PI*2);
                ctx.fill();
                ctx.shadowBlur = 0; 
                
                ctx.beginPath();
                ctx.strokeStyle = `rgba(0, 255, 85, ${rAlpha * 0.6})`;
                ctx.lineWidth = 1.5 * r.proj.scale;
                let tailP = new Point3D(
                    Math.cos(r.angle - r.speed * 15) * r.dist,
                    r.yOff,
                    Math.sin(r.angle - r.speed * 15) * r.dist
                );
                tailP.rotateX(r.rx); tailP.rotateZ(r.rz);
                let tProj = tailP.project(fov, viewDist);
                ctx.moveTo(r.proj.px, r.proj.py);
                ctx.lineTo(tProj.px, tProj.py);
                ctx.stroke();
            }
        });
    }

    // --- RENDER LAYERS FOR PERFECT 3D OCCLUSION ---
    
    // LAYER 1: BACK HALF OF GLOBE & ROCKETS
    drawGlobe(false);
    drawRockets(false);

    // LAYER 2: THE PICTURE DEAD CENTER
    if(profileImg.complete) {
        ctx.save();
        ctx.beginPath();
        let imgRad = 95; // Radius of profile picture
        ctx.arc(cx, cy, imgRad, 0, Math.PI*2);
        ctx.clip();
        
        ctx.filter = "grayscale(70%) contrast(1.2) brightness(0.9)";
        ctx.drawImage(profileImg, cx - imgRad, cy - imgRad, imgRad*2, imgRad*2);
        ctx.restore();
        
        // Solid Glowing Ring Border
        ctx.beginPath();
        ctx.arc(cx, cy, imgRad, 0, Math.PI*2);
        ctx.strokeStyle = "rgba(0, 255, 85, 0.9)";
        ctx.lineWidth = 3;
        ctx.shadowBlur = 20;
        ctx.shadowColor = "rgba(0, 255, 85, 1)";
        ctx.stroke();
        ctx.shadowBlur = 0;
    }

    // LAYER 3: FRONT HALF OF GLOBE & ROCKETS
    drawGlobe(true);
    drawRockets(true);

    requestAnimationFrame(renderSpace);
}
renderSpace();

const lines = [
    "INITIALIZING_CONNECTION...",
    "FETCHING_DATA: ML_MODELS // PYTHON",
    "STATUS: OPERATIONAL",
    "DECRYPTING_BIO...",
    "TARGETING Skit-025"
];
let lIdx = 0, chIdx = 0;
function typeEth() {
    const el = document.getElementById('target-ethereal');
    if (lIdx >= lines.length) {
        lIdx = 0; 
        setTimeout(typeEth, 5000); 
        return;
    }
    const currentLine = lines[lIdx];
    el.textContent = currentLine.substring(0, chIdx);
    chIdx++;
    if (chIdx > currentLine.length) {
        chIdx = 0;
        lIdx++;
        setTimeout(typeEth, 1000);
        return;
    }
    setTimeout(typeEth, Math.random() * 50 + 20); 
}
setTimeout(typeEth, 1000);
</script>

<br>

# 💫 About Me:
I am a Computer Science and Engineering student 👨💻, driven by curiosity and a passion for coding 💡. As a Python developer 🐍 and AI/ML Engineer 🤖, I am constantly exploring new ways to apply technology to real-world problems 🌍. Currently, I am strengthening my foundation in Data Structures and Algorithms 📊 to refine my problem-solving skills.<br><br>For me, coding is more than just a skill—it’s my passion steadily evolving into a profession 🚀. I focus on giving practical solutions to the world by my projects🛠️ through consistent implementation, experimentation, and refinement 🔄. My mindset is shaped by continuous learning 📈, adaptability, and the drive to improve with every challenge.<br><br>I believe in learning and growing together 🤝, and I am always open to collaboration, knowledge-sharing, and pushing boundaries to create impactful innovations ✨.


## 🌐 Socials:
[![Instagram](https://img.shields.io/badge/Instagram-%23E4405F.svg?logo=Instagram&logoColor=white)](https://instagram.com/skit025) [![LinkedIn](https://img.shields.io/badge/LinkedIn-%230077B5.svg?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/aditya-prasad-barik-838849378?utm_source=share_via&utm_content=profile&utm_medium=member_android) [![email](https://img.shields.io/badge/Email-D14836?logo=gmail&logoColor=white)](mailto:codesheritage@gmail.com) 

# 💻 Tech Stack:
![C](https://img.shields.io/badge/c-%2300599C.svg?style=flat-square&logo=c&logoColor=white) ![CSS3](https://img.shields.io/badge/css3-%231572B6.svg?style=flat-square&logo=css3&logoColor=white) ![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=flat-square&logo=openjdk&logoColor=white) ![HTML5](https://img.shields.io/badge/html5-%23E34F26.svg?style=flat-square&logo=html5&logoColor=white) ![Python](https://img.shields.io/badge/python-3670A0?style=flat-square&logo=python&logoColor=ffdd54) ![PowerShell](https://img.shields.io/badge/PowerShell-%235391FE.svg?style=flat-square&logo=powershell&logoColor=white) ![Bash Script](https://img.shields.io/badge/bash_script-%23121011.svg?style=flat-square&logo=gnu-bash&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=flat-square&logo=amazon-aws&logoColor=white) ![Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=flat-square&logo=microsoftazure&logoColor=white) ![Vercel](https://img.shields.io/badge/vercel-%23000000.svg?style=flat-square&logo=vercel&logoColor=white) ![Netlify](https://img.shields.io/badge/netlify-%23000000.svg?style=flat-square&logo=netlify&logoColor=#00C7B7) ![Anaconda](https://img.shields.io/badge/Anaconda-%2344A833.svg?style=flat-square&logo=anaconda&logoColor=white) ![nVIDIA](https://img.shields.io/badge/cuda-000000.svg?style=flat-square&logo=nVIDIA&logoColor=green) ![Django](https://img.shields.io/badge/django-%23092E20.svg?style=flat-square&logo=django&logoColor=white) ![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat-square&logo=fastapi) ![Flask](https://img.shields.io/badge/flask-%23000.svg?style=flat-square&logo=flask&logoColor=white) ![OpenCV](https://img.shields.io/badge/opencv-%23white.svg?style=flat-square&logo=opencv&logoColor=white) ![NPM](https://img.shields.io/badge/NPM-%23CB3837.svg?style=flat-square&logo=npm&logoColor=white) ![Socket.io](https://img.shields.io/badge/Socket.io-black?style=flat-square&logo=socket.io&badgeColor=010101) ![Apache Tomcat](https://img.shields.io/badge/apache%20tomcat-%23F8DC75.svg?style=flat-square&logo=apache-tomcat&logoColor=black) ![MySQL](https://img.shields.io/badge/mysql-4479A1.svg?style=flat-square&logo=mysql&logoColor=white) ![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=flat-square&logo=postgresql&logoColor=white) ![MongoDB](https://img.shields.io/badge/MongoDB-%234ea94b.svg?style=flat-square&logo=mongodb&logoColor=white) ![Canva](https://img.shields.io/badge/Canva-%2300C4CC.svg?style=flat-square&logo=Canva&logoColor=white) ![Blender](https://img.shields.io/badge/blender-%23F5792A.svg?style=flat-square&logo=blender&logoColor=white) ![Keras](https://img.shields.io/badge/Keras-%23D00000.svg?style=flat-square&logo=Keras&logoColor=white) ![Matplotlib](https://img.shields.io/badge/Matplotlib-%23ffffff.svg?style=flat-square&logo=Matplotlib&logoColor=black) ![NumPy](https://img.shields.io/badge/numpy-%23013243.svg?style=flat-square&logo=numpy&logoColor=white) ![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=flat-square&logo=pandas&logoColor=white) ![PyTorch](https://img.shields.io/badge/PyTorch-%23EE4C2C.svg?style=flat-square&logo=PyTorch&logoColor=white) ![scikit-learn](https://img.shields.io/badge/scikit--learn-%23F7931E.svg?style=flat-square&logo=scikit-learn&logoColor=white) ![TensorFlow](https://img.shields.io/badge/TensorFlow-%23FF6F00.svg?style=flat-square&logo=TensorFlow&logoColor=white) ![Scipy](https://img.shields.io/badge/SciPy-%230C55A5.svg?style=flat-square&logo=scipy&logoColor=%white) ![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=flat-square&logo=github&logoColor=white) ![Git](https://img.shields.io/badge/git-%23F05033.svg?style=flat-square&logo=git&logoColor=white) ![Twilio](https://img.shields.io/badge/Twilio-F22F46?style=flat-square&logo=Twilio&logoColor=white) ![nVIDIA](https://img.shields.io/badge/nVIDIA-%2376B900.svg?style=flat-square&logo=nVIDIA&logoColor=white) ![AMD](https://img.shields.io/badge/AMD-%23000000.svg?style=flat-square&logo=amd&logoColor=white)
# 📊 GitHub Stats:
![](https://github-readme-stats.shion.dev/api?username=Skit-025&theme=blue_navy&hide_border=true&include_all_commits=false&count_private=false)<br/>
![](https://streak-stats.demolab.com/?user=Skit-025&theme=blue_navy&hide_border=true)<br/>
![](https://github-readme-stats.shion.dev/api/top-langs/?username=Skit-025&theme=blue_navy&hide_border=true&include_all_commits=false&count_private=false&layout=compact)

## 🏆 GitHub Trophies
![](https://github-profile-trophy.vercel.app/?username=Skit-025&theme=blue_navy&no-frame=false&no-bg=true&margin-w=4)

### 🔝 Top Contributed Repo
![](https://github-contributor-stats.vercel.app/api?username=Skit-025&limit=5&theme=dark&combine_all_yearly_contributions=true)

---
[![](https://visitcountpro.netlify.app/api?id=Skit-025&pretty=true)](https://visitcount.itsvg.in)

#Thankyou for visiting my Profile❤️🤖
![snake gif](https://github.com/Skit-025/Skit-025/blob/output/github-snake-dark.svg)
</div>
