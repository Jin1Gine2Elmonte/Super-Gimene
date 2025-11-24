<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Resonant Field | التناغم الكوني</title>
    <style>
        :root {
            --bg-color: #020406;
            --text-color: rgba(255, 255, 255, 0.7);
            --accent-color: #ffd700;
        }
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: var(--bg-color);
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            cursor: crosshair;
        }
        canvas {
            display: block;
        }
        .ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            z-index: 10;
        }
        .start-btn {
            pointer-events: auto;
            background: transparent;
            border: 1px solid rgba(255, 255, 255, 0.3);
            color: white;
            padding: 15px 40px;
            font-size: 1.1rem;
            letter-spacing: 2px;
            text-transform: uppercase;
            cursor: pointer;
            transition: all 0.5s ease;
            backdrop-filter: blur(5px);
            border-radius: 30px;
        }
        .start-btn:hover {
            background: rgba(255, 255, 255, 0.1);
            border-color: var(--accent-color);
            box-shadow: 0 0 20px rgba(255, 215, 0, 0.2);
        }
        .overlay-text {
            position: absolute;
            bottom: 30px;
            text-align: center;
            opacity: 0.6;
            font-size: 0.8rem;
            letter-spacing: 1px;
            transition: opacity 1s;
        }
        .hidden {
            opacity: 0;
            pointer-events: none;
        }
    </style>
</head>
<body>

    <div class="ui-layer" id="ui">
        <button class="start-btn" id="startBtn">ابدأ الرحلة</button>
        <div class="overlay-text">
            السكون يخلق النظام • الحركة تخلق الحياة
            <br>
            <span style="font-size: 0.7em; opacity: 0.7">استخدم السماعات لأفضل تجربة حسية</span>
        </div>
    </div>

    <canvas id="canvas"></canvas>

<script>
/**
 * THE RESONANT FIELD - التناغم الكوني
 * ===================================
 * هذا الكود يمزج بين ثلاث مدارس:
 * 1. الهندسة المقدسة (Sacred Geometry): استخدام معادلات فيبوناتشي لتحديد "الموطن الأصلي" للجسيمات.
 * 2. ديناميكا السوائل (Fluid Dynamics): استخدام حقول الضجيج لمحاكاة العاطفة والتدفق.
 * 3. التوليف الصوتي (Sonic Synthesis): استخدام Web Audio API لترجمة الحركة إلى ترددات صوتية علاجية.
 * * الفلسفة:
 * الجسيمات لها "ذاكرة" لمكانها الهندسي المقدس، لكنها تملك "حرية" الانجراف مع تيار الشعور.
 * عندما يتوقف المستخدم (السكون)، يعود النظام (الماندالا).
 * عندما يتحرك المستخدم (الفوضى الخلاقة)، يذوب النظام ليصبح سائلاً.
 */

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const ui = document.getElementById('ui');
const startBtn = document.getElementById('startBtn');

// إعدادات الكانفاس
let width, height, centerX, centerY;
let particles = [];
const particleCount = 1800; // عدد الأرواح
const PHI = (1 + Math.sqrt(5)) / 2; // النسبة الذهبية
const GOLDEN_ANGLE = 2 * Math.PI * (1 - 1/PHI); // الزاوية الذهبية (~137.5 درجة)

// المتغيرات الفيزيائية
let time = 0;
let chaosLevel = 0; // 0 = نظام تام، 1 = فوضى تامة
let targetChaos = 0;

// الماوس
let mouse = { x: 0, y: 0, speed: 0, active: false };
let lastMouse = { x: 0, y: 0 };

// الصوت
let audioCtx;
let oscillator;
let gainNode;
let filterNode;
let audioStarted = false;

// تهيئة النظام
function resize() {
    width = canvas.width = window.innerWidth;
    height = canvas.height = window.innerHeight;
    centerX = width / 2;
    centerY = height / 2;
}

// === محرك الصوت (Audio Engine) ===
// يولد نغمة "Drone" عميقة تتغير حدتها مع فوضى النظام
function initAudio() {
    const AudioContext = window.AudioContext || window.webkitAudioContext;
    audioCtx = new AudioContext();
    
    // المذبذب الأساسي (Carrier)
    oscillator = audioCtx.createOscillator();
    oscillator.type = 'sine';
    oscillator.frequency.value = 110; // تردد A2 (عميق ومهدئ)

    // فلتر (Lowpass) لجعل الصوت ناعماً
    filterNode = audioCtx.createBiquadFilter();
    filterNode.type = 'lowpass';
    filterNode.frequency.value = 200;

    // التحكم في الصوت (Gain)
    gainNode = audioCtx.createGain();
    gainNode.gain.value = 0.0; // نبدأ بصمت

    // ربط العقد
    oscillator.connect(filterNode);
    filterNode.connect(gainNode);
    gainNode.connect(audioCtx.destination);

    oscillator.start();
    audioStarted = true;
    
    // Fade in
    gainNode.gain.setTargetAtTime(0.15, audioCtx.currentTime, 2);
}

function updateAudio() {
    if (!audioStarted) return;
    
    // تغيير التردد بناءً على مستوى الفوضى (Chaos)
    // الفوضى ترفع التردد قليلاً وتفتح الفلتر
    const targetFreq = 110 + (chaosLevel * 50); // من 110 هرتز إلى 160 هرتز
    const targetFilter = 200 + (chaosLevel * 800); // الفلتر يفتح عند الحركة
    
    oscillator.frequency.setTargetAtTime(targetFreq, audioCtx.currentTime, 0.1);
    filterNode.frequency.setTargetAtTime(targetFilter, audioCtx.currentTime, 0.1);
    
    // Panning (توزيع الصوت ستيريو وهمي بسيط عبر تغيير التردد الطفيف جداً) يمكن إضافته لاحقاً
}

// === فئة الروح (Particle Class) ===
class Particle {
    constructor(index) {
        this.index = index;
        this.pos = { x: centerX, y: centerY };
        this.vel = { x: 0, y: 0 };
        this.acc = { x: 0, y: 0 };
        this.size = Math.random() * 2 + 0.5;
        this.hue = 0;
        
        // حساب الموقع "المقدس" (Sacred Target) بناءً على فيبوناتشي
        // R = c * sqrt(n), Theta = n * GoldenAngle
        const spacing = 4; // مسافة التباعد
        const angle = this.index * GOLDEN_ANGLE;
        const radius = spacing * Math.sqrt(this.index);
        
        // الموضع المستهدف بالنسبة للمركز
        this.homeParams = { angle, radius };
    }

    getHomePosition() {
        // ندور الماندالا ببطء مع الزمن
        const currentAngle = this.homeParams.angle + time * 0.05;
        const tx = centerX + Math.cos(currentAngle) * this.homeParams.radius * (1 + Math.sin(time * 0.2)*0.05);
        const ty = centerY + Math.sin(currentAngle) * this.homeParams.radius * (1 + Math.sin(time * 0.2)*0.05);
        return { x: tx, y: ty };
    }

    update() {
        const home = this.getHomePosition();
        
        // 1. حساب قوة التدفق (Flow Force) - تمثل العاطفة/الفوضى
        // ضجيج يعتمد على الموقع
        const noiseScale = 0.003;
        const angleNoise = (Math.sin(this.pos.x * noiseScale + time) + Math.cos(this.pos.y * noiseScale)) * Math.PI;
        
        const flowForce = {
            x: Math.cos(angleNoise),
            y: Math.sin(angleNoise)
        };

        // 2. حساب قوة العودة للمنزل (Home Force) - تمثل الروح/النظام
        const dx = home.x - this.pos.x;
        const dy = home.y - this.pos.y;
        const distToHome = Math.sqrt(dx*dx + dy*dy);
        
        const homeForce = {
            x: dx * 0.05, // جاذبية مرنة
            y: dy * 0.05
        };

        // 3. المزج بين القوتين بناءً على مستوى الفوضى العالمي
        // إذا كان المستخدم يتحرك (chaosLevel عالي) -> نتبع التدفق
        // إذا كان المستخدم ساكناً (chaosLevel منخفض) -> نتبع النظام
        
        this.acc.x += (homeForce.x * (1 - chaosLevel)) + (flowForce.x * chaosLevel * 0.5);
        this.acc.y += (homeForce.y * (1 - chaosLevel)) + (flowForce.y * chaosLevel * 0.5);

        // 4. تأثير الماوس (اضطراب)
        if (mouse.speed > 0.1) {
            const mdx = mouse.x - this.pos.x;
            const mdy = mouse.y - this.pos.y;
            const mDist = Math.sqrt(mdx*mdx + mdy*mdy);
            
            if (mDist < 200) {
                // دفع الجسيمات بعيداً عند الحركة السريعة
                const push = (200 - mDist) / 200;
                this.acc.x -= (mdx / mDist) * push * mouse.speed * 0.5;
                this.acc.y -= (mdy / mDist) * push * mouse.speed * 0.5;
            }
        }

        // الفيزياء
        this.vel.x += this.acc.x;
        this.vel.y += this.acc.y;
        
        // كبح السرعة (Damping)
        this.vel.x *= 0.92;
        this.vel.y *= 0.92;

        this.pos.x += this.vel.x;
        this.pos.y += this.vel.y;
        this.acc.x = 0; this.acc.y = 0;

        // حساب اللون
        // النظام (المركز) = ذهبي/برتقالي
        // الفوضى (الأطراف) = أزرق/بنفسجي
        const speed = Math.sqrt(this.vel.x*this.vel.x + this.vel.y*this.vel.y);
        const distFromCenter = Math.sqrt((this.pos.x-centerX)**2 + (this.pos.y-centerY)**2);
        
        // معادلة لونية تمزج بين الروحانية (الذهبي) والعمق (الأزرق)
        const baseHue = 200; // أزرق
        const targetHue = 45; // ذهبي
        
        // كلما اقتربنا من "المنزل" والنظام، أصبح اللون ذهبياً أكثر
        // كلما زادت السرعة والبعد، أصبح أزرق سماوياً
        const mix = Math.min(1, (1 - chaosLevel) + (speed * 0.1));
        this.hue = baseHue + (targetHue - baseHue) * (1 - chaosLevel); // يتغير اللون مع النظام العام
        
        // اللمعان يعتمد على السرعة
        this.lightness = 50 + speed * 10;
    }

    draw() {
        ctx.beginPath();
        // رسم خط قصير يمثل الحركة (Trail بسيط)
        ctx.moveTo(this.pos.x - this.vel.x * 2, this.pos.y - this.vel.y * 2);
        ctx.lineTo(this.pos.x, this.pos.y);
        
        // الشفافية تتغير: النظام شفاف أكثر، الفوضى واضحة أكثر
        const alpha = 0.3 + (chaosLevel * 0.4);
        
        ctx.strokeStyle = `hsla(${this.hue}, 80%, ${this.lightness}%, ${alpha})`;
        ctx.lineWidth = this.size;
        ctx.lineCap = 'round';
        ctx.stroke();
    }
}

function init() {
    resize();
    particles = [];
    for (let i = 0; i < particleCount; i++) {
        particles.push(new Particle(i));
    }
}

function animate() {
    // تحديث الزمن
    time += 0.01;
    
    // تلاشي الماوس البطيء
    mouse.speed *= 0.95;
    if (mouse.speed < 0.01) mouse.speed = 0;
    
    // منطق الفوضى مقابل النظام
    // إذا تحرك الماوس بسرعة، ترتفع الفوضى. إذا توقف، تهبط تدريجياً للصفر (النظام)
    targetChaos = Math.min(1, mouse.speed * 0.2); 
    // نعومة الانتقال بين الحالتين (Interpolation)
    chaosLevel += (targetChaos - chaosLevel) * 0.05;

    // رسم الخلفية مع أثر (Trail)
    // نستخدم لوناً داكناً جداً للمسح ليعطي إيحاءً بالعمق
    ctx.fillStyle = 'rgba(2, 4, 6, 0.2)'; 
    ctx.fillRect(0, 0, width, height);

    ctx.globalCompositeOperation = 'lighter'; // مزج ضوئي

    particles.forEach(p => {
        p.update();
        p.draw();
    });
    
    ctx.globalCompositeOperation = 'source-over';
    
    // تحديث الصوت
    updateAudio();

    requestAnimationFrame(animate);
}

// === إدارة الأحداث ===

// التعامل مع الماوس لحساب السرعة
window.addEventListener('mousemove', e => {
    const dx = e.clientX - lastMouse.x;
    const dy = e.clientY - lastMouse.y;
    // سرعة الماوس الحالية
    const currentSpeed = Math.sqrt(dx*dx + dy*dy);
    
    mouse.x = e.clientX;
    mouse.y = e.clientY;
    mouse.speed = currentSpeed; // نخزن السرعة للتأثير على الفوضى
    
    lastMouse.x = e.clientX;
    lastMouse.y = e.clientY;
});

// اللمس
window.addEventListener('touchmove', e => {
    const t = e.touches[0];
    const dx = t.clientX - lastMouse.x;
    const dy = t.clientY - lastMouse.y;
    const currentSpeed = Math.sqrt(dx*dx + dy*dy);
    
    mouse.x = t.clientX;
    mouse.y = t.clientY;
    mouse.speed = currentSpeed;
    
    lastMouse.x = t.clientX;
    lastMouse.y = t.clientY;
    e.preventDefault();
}, {passive: false});

window.addEventListener('resize', resize);

// زر البدء (مطلوب لتشغيل الصوت في المتصفحات الحديثة)
startBtn.addEventListener('click', () => {
    initAudio();
    ui.classList.add('hidden');
    setTimeout(() => {
        ui.style.display = 'none';
    }, 1000);
});

// التشغيل الأولي
init();
animate();

</script>
</body>
</html>

