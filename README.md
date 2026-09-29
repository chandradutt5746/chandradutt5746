from PIL import Image, ImageDraw, ImageFont, ImageFilter
import os, math, random, textwrap, zipfile, shutil

base = "/mnt/data/chandradutt-github-profile"
assets = os.path.join(base, "assets")
os.makedirs(assets, exist_ok=True)

# ---------- Fonts ----------
font_candidates = [
    "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf",
    "/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf",
]
mono_candidates = [
    "/usr/share/fonts/truetype/dejavu/DejaVuSansMono.ttf",
    "/usr/share/fonts/truetype/dejavu/DejaVuSansMono-Bold.ttf",
]
bold_candidates = [
    "/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf",
]
def pick(cands):
    for p in cands:
        if os.path.exists(p):
            return p
    return None

FONT = pick(font_candidates)
BOLD = pick(bold_candidates)
MONO = pick(mono_candidates)

def font(path, size):
    return ImageFont.truetype(path, size)

# ---------- Palette ----------
BG = (7, 11, 19)
BG2 = (10, 17, 29)
CYAN = (70, 220, 255)
CYAN2 = (38, 160, 210)
VIOLET = (145, 105, 255)
WHITE = (239, 247, 255)
MUTED = (145, 165, 185)
GREEN = (70, 235, 165)
LINE = (30, 55, 78)

random.seed(42)

def rounded(draw, box, radius=18, fill=None, outline=None, width=1):
    draw.rounded_rectangle(box, radius=radius, fill=fill, outline=outline, width=width)

def draw_glow(base_img, layer):
    glow = layer.filter(ImageFilter.GaussianBlur(14))
    base_img.alpha_composite(glow)

# ---------- Hero GIF ----------
W, H = 1200, 430
frames = []
stars = [(random.randint(30, W-30), random.randint(25, H-25), random.randint(1, 2), random.random()*math.tau) for _ in range(100)]

for f in range(36):
    im = Image.new("RGBA", (W, H), BG + (255,))
    d = ImageDraw.Draw(im, "RGBA")

    # subtle background gradient bands
    for y in range(H):
        t = y / H
        c = tuple(int(BG[i]*(1-t) + BG2[i]*t) for i in range(3)) + (255,)
        d.line((0, y, W, y), fill=c)

    # grid
    for x in range(0, W, 48):
        d.line((x, 0, x, H), fill=LINE + (45,), width=1)
    for y in range(0, H, 48):
        d.line((0, y, W, y), fill=LINE + (35,), width=1)

    # stars / particles
    for x, y, r, ph in stars:
        alpha = int(40 + 45*(0.5 + 0.5*math.sin(f*0.25 + ph)))
        d.ellipse((x-r, y-r, x+r, y+r), fill=CYAN + (alpha,))

    # circuit lines
    nodes = [(110, 105), (205, 210), (1085, 125), (1000, 295), (160, 335), (1040, 365)]
    for idx, (nx, ny) in enumerate(nodes):
        cx = W//2
        cy = 258
        # line toward central area
        d.line((nx, ny, cx, cy), fill=CYAN2 + (60,), width=2)
        d.ellipse((nx-4, ny-4, nx+4, ny+4), fill=CYAN + (170,))
    pulse = (math.sin(f*0.45)+1)/2
    for x, y, r, ph in stars[:18]:
        rr = int(2 + 4*pulse)
        d.ellipse((x-rr, y-rr, x+rr, y+rr), fill=VIOLET + (75,))

    # central hexagonal "secure compute" emblem
    cx, cy = 600, 220
    rad = 78
    pts = []
    for i in range(6):
        a = math.radians(60*i - 30)
        pts.append((cx + rad*math.cos(a), cy + rad*math.sin(a)))
    glow_layer = Image.new("RGBA", (W, H), (0,0,0,0))
    gd = ImageDraw.Draw(glow_layer, "RGBA")
    gd.polygon(pts, outline=CYAN + (150,), width=8)
    draw_glow(im, glow_layer)
    d = ImageDraw.Draw(im, "RGBA")
    d.polygon(pts, outline=CYAN + (230,), width=3)

    # lock
    d.rounded_rectangle((575, 215, 625, 263), radius=8, fill=BG2+(240,), outline=WHITE+(200,), width=2)
    d.arc((586, 191, 614, 232), 180, 360, fill=WHITE+(220,), width=4)
    d.ellipse((597, 233, 603, 241), fill=CYAN+(235,))
    d.line((600, 240, 600, 253), fill=CYAN+(235,), width=3)

    # moving ciphertext packets
    for i in range(12):
        t = ((f*0.035 + i/12) % 1)
        x = int(320 + t*560)
        y = int(155 + 105*math.sin(i*1.7 + f*0.12))
        block = 5 + (i % 3)
        d.rounded_rectangle((x, y, x+block*3, y+7), radius=2, fill=CYAN+(130,))
    
    # title
    title = "CHANDRADUTT PATEL"
    subtitle = "SOFTWARE ENGINEER  ·  PRIVACY  ·  CRYPTOGRAPHY"
    tfont = font(BOLD, 34)
    sfont = font(MONO, 16)
    tw = d.textbbox((0,0), title, font=tfont)[2]
    sw = d.textbbox((0,0), subtitle, font=sfont)[2]
    d.text(((W-tw)/2, 35), title, font=tfont, fill=WHITE+(255,))
    d.text(((W-sw)/2, 80), subtitle, font=sfont, fill=CYAN+(230,))

    # tagline
    tagline = "Building secure systems where sensitive data can remain protected."
    tf = font(FONT, 15)
    tw = d.textbbox((0,0), tagline, font=tf)[2]
    d.text(((W-tw)/2, 370), tagline, font=tf, fill=MUTED+(225,))

    frames.append(im.convert("P", palette=Image.ADAPTIVE, colors=256))

hero_path = os.path.join(assets, "hero.gif")
frames[0].save(hero_path, save_all=True, append_images=frames[1:], duration=90, loop=0, optimize=True)

# ---------- FHE Flow GIF ----------
W2, H2 = 1200, 310
flow_frames = []
for f in range(32):
    im = Image.new("RGBA", (W2, H2), BG+(255,))
    d = ImageDraw.Draw(im, "RGBA")
    for x in range(0, W2, 60):
        d.line((x, 0, x, H2), fill=LINE+(25,), width=1)
    for y in range(0, H2, 50):
        d.line((0, y, W2, y), fill=LINE+(20,), width=1)

    boxes = [
        (70, 105, 285, 205, "SENSITIVE DATA", CYAN),
        (365, 105, 580, 205, "ENCRYPT", VIOLET),
        (660, 75, 880, 235, "SECURE COMPUTE", GREEN),
        (960, 105, 1130, 205, "RESULT", CYAN)
    ]
    for x1,y1,x2,y2,label,col in boxes:
        rounded(d, (x1,y1,x2,y2), 18, fill=(12,20,33,220), outline=col+(160,), width=2)
        lf = font(BOLD if label!="SECURE COMPUTE" else MONO, 16)
        tw = d.textbbox((0,0), label, font=lf)[2]
        d.text(((x1+x2-tw)/2, y1+34), label, font=lf, fill=WHITE+(235,))

    # arrows
    arrows = [(285,155,365,155), (580,155,660,155), (880,155,960,155)]
    for ax1,ay1,ax2,ay2 in arrows:
        d.line((ax1,ay1,ax2,ay2), fill=MUTED+(145,), width=3)
        d.polygon([(ax2,ay2),(ax2-12,ay2-7),(ax2-12,ay2+7)], fill=MUTED+(180,))

    # packet movement through pipeline
    for i in range(18):
        t = ((f*0.04 + i/18) % 1)
        x = 160 + t*850
        y = 155 + math.sin(i*0.8 + f*0.13)*4
        c = CYAN if i % 2 == 0 else VIOLET
        d.rounded_rectangle((x-9,y-5,x+9,y+5), radius=3, fill=c+(170,))

    # micro labels
    mf = font(MONO, 12)
    d.text((78, 230), "plaintext", font=mf, fill=MUTED+(200,))
    d.text((372, 230), "ciphertext", font=mf, fill=MUTED+(200,))
    d.text((682, 248), "compute without decryption", font=mf, fill=GREEN+(210,))
    d.text((972, 230), "prediction", font=mf, fill=MUTED+(200,))

    flow_frames.append(im.convert("P", palette=Image.ADAPTIVE, colors=256))

flow_path = os.path.join(assets, "fhe-flow.gif")
flow_frames[0].save(flow_path, save_all=True, append_images=flow_frames[1:], duration=100, loop=0, optimize=True)

# ---------- Footer GIF ----------
W3, H3 = 1200, 150
footer_frames = []
for f in range(24):
    im = Image.new("RGBA", (W3,H3), BG+(255,))
    d = ImageDraw.Draw(im, "RGBA")
    # waves
    for k, amp in enumerate([16, 10, 6]):
        pts = []
        for x in range(0, W3+20, 10):
            y = 70 + k*9 + amp*math.sin(x/100 + f*0.22 + k)
            pts.append((x,y))
        d.line(pts, fill=(CYAN if k==0 else VIOLET)+(90-k*20,), width=2)
    text = "BUILD • SECURE • COMPUTE • REPEAT"
    tf = font(BOLD, 18)
    tw = d.textbbox((0,0), text, font=tf)[2]
    d.text(((W3-tw)/2, 35), text, font=tf, fill=WHITE+(235,))
    sf = font(MONO, 12)
    sub = "Python  ·  C++  ·  Cloud  ·  Data  ·  FHE"
    sw = d.textbbox((0,0), sub, font=sf)[2]
    d.text(((W3-sw)/2, 91), sub, font=sf, fill=MUTED+(210,))
    footer_frames.append(im.convert("P", palette=Image.ADAPTIVE, colors=256))

footer_path = os.path.join(assets, "footer.gif")
footer_frames[0].save(footer_path, save_all=True, append_images=footer_frames[1:], duration=110, loop=0, optimize=True)

# ---------- Static architecture SVG ----------
architecture_svg = """<svg width="1200" height="360" viewBox="0 0 1200 360" xmlns="http://www.w3.org/2000/svg">
<rect width="1200" height="360" rx="28" fill="#070B13"/>
<g opacity=".35" stroke="#1E374E">
  <path d="M0 60H1200M0 120H1200M0 180H1200M0 240H1200M0 300H1200"/>
  <path d="M80 0V360M160 0V360M240 0V360M320 0V360M400 0V360M480 0V360M560 0V360M640 0V360M720 0V360M800 0V360M880 0V360M960 0V360M1040 0V360M1120 0V360"/>
</g>
<g font-family="Arial, sans-serif" text-anchor="middle">
  <rect x="70" y="118" width="220" height="124" rx="20" fill="#0C1421" stroke="#46DCFF"/>
  <text x="180" y="158" fill="#EFF7FF" font-size="19" font-weight="700">SENSITIVE DATA</text>
  <text x="180" y="192" fill="#91A5B9" font-size="14">Healthcare • Finance • AI</text>
  <text x="180" y="216" fill="#91A5B9" font-size="14">Private input</text>

  <rect x="365" y="92" width="210" height="176" rx="20" fill="#0C1421" stroke="#9169FF"/>
  <text x="470" y="135" fill="#EFF7FF" font-size="19" font-weight="700">ENCRYPTION</text>
  <text x="470" y="169" fill="#91A5B9" font-size="14">BFV • BGV • CKKS</text>
  <text x="470" y="195" fill="#91A5B9" font-size="14">Keys remain protected</text>
  <text x="470" y="230" fill="#46DCFF" font-size="14">Ciphertext →</text>

  <rect x="650" y="72" width="230" height="216" rx="20" fill="#0C1421" stroke="#46EBA5"/>
  <text x="765" y="118" fill="#EFF7FF" font-size="19" font-weight="700">SECURE COMPUTE</text>
  <text x="765" y="153" fill="#91A5B9" font-size="14">ML • Analytics • APIs</text>
  <text x="765" y="181" fill="#91A5B9" font-size="14">Compute on ciphertext</text>
  <text x="765" y="223" fill="#46EBA5" font-size="15" font-weight="700">NO PLAINTEXT EXPOSED</text>

  <rect x="950" y="118" width="180" height="124" rx="20" fill="#0C1421" stroke="#46DCFF"/>
  <text x="1040" y="158" fill="#EFF7FF" font-size="19" font-weight="700">RESULT</text>
  <text x="1040" y="192" fill="#91A5B9" font-size="14">Prediction</text>
  <text x="1040" y="216" fill="#91A5B9" font-size="14">Protected output</text>
</g>
<g stroke="#91A5B9" stroke-width="3" fill="none">
  <path d="M290 180H365"/><path d="M575 180H650"/><path d="M880 180H950"/>
</g>
<g fill="#91A5B9">
  <path d="M365 180l-12-7v14z"/><path d="M650 180l-12-7v14z"/><path d="M950 180l-12-7v14z"/>
</g>
</svg>"""
with open(os.path.join(assets, "architecture.svg"), "w", encoding="utf-8") as f:
    f.write(architecture_svg)

# ---------- README ----------
readme = r'''<div align="center">

<img src="./assets/hero.gif" alt="Chandradutt Patel — Software Engineer, Privacy, Cryptography" width="100%"/>

<br/>

<a href="https://github.com/chandradutt5746">
  <img src="https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white" alt="GitHub"/>
</a>
<a href="https://linkedin.com/in/cnpatel5746">
  <img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white" alt="LinkedIn"/>
</a>
<a href="mailto:cnpatel5746@gmail.com">
  <img src="https://img.shields.io/badge/Email-EA4335?style=flat-square&logo=gmail&logoColor=white" alt="Email"/>
</a>

<br/><br/>

<em>Secure systems • privacy-preserving computation • software engineering</em>

</div>

---

## `01` / who I am

I’m a **Software Engineer** working across backend development, APIs, cloud infrastructure and data processing, with a growing focus on **privacy-preserving computation**.

I enjoy the space where difficult engineering problems meet sensitive data:

> **How do we build useful software without unnecessarily exposing the data it operates on?**

That question has led me from Python backend systems and cloud-native applications into **Fully Homomorphic Encryption (FHE)**, secure computation and privacy-preserving machine learning.

### My engineering focus

| Area | What I build / explore |
|---|---|
| ⚙️ Software Engineering | Python services, APIs, microservices, testing, CI/CD |
| ☁️ Cloud & Infrastructure | AWS, containers, Terraform, deployment automation |
| 📊 Data & ML | PySpark, Databricks, machine learning, data workflows |
| 🔐 Privacy & Cryptography | FHE, Microsoft SEAL, OpenFHE, encrypted computation |
| 🧩 Systems | Python ↔ C++ integration, performance-oriented native components |

---

## `02` / what I'm building

<div align="center">

### 🔐 Privacy-preserving computation

<img src="./assets/fhe-flow.gif" alt="Encrypted data flowing through secure computation" width="100%"/>

</div>

The idea is simple:

**data is valuable — and sometimes too sensitive to expose during computation.**

My current work explores systems where computation can happen on protected data, combining:

`Python` `C++` `FHE` `Machine Learning` `Cloud Infrastructure`

### A simplified architecture

<div align="center">

<img src="./assets/architecture.svg" alt="Privacy-preserving computation architecture" width="100%"/>

</div>

---

## `03` / featured work

### 🔐 SEAL-PYTHON

**Python ↔ C++ integration around Microsoft SEAL**

<a href="https://github.com/chandradutt5746/SEAL-PYTHON-4.1.5">
<img src="https://img.shields.io/badge/OPEN_REPOSITORY-46DCFF?style=for-the-badge&logo=github&logoColor=07101A" alt="Open repository"/>
</a>

A hands-on project exploring practical Python access to Microsoft SEAL through native C++ bindings.

**Focus**

`Microsoft SEAL` · `C++` · `Python` · `Pybind11` · `BFV` · `BGV` · `CKKS`

---

### 🧪 OpenFHE Python API

**Python experimentation and testing around OpenFHE**

<a href="https://github.com/chandradutt5746/openfhe-python-api">
<img src="https://img.shields.io/badge/OPEN_REPOSITORY-9169FF?style=for-the-badge&logo=github&logoColor=white" alt="Open repository"/>
</a>

A practical workspace for exploring OpenFHE APIs, encrypted computation workflows and reproducible examples.

**Focus**

`OpenFHE` · `Python` · `FHE` · `BFV` · `BGV` · `CKKS` · `Testing`

---

### 🌐 Portfolio Website

**Full-stack application built around a Django backend**

<a href="https://github.com/chandradutt5746/Portfoliowebsite">
<img src="https://img.shields.io/badge/OPEN_REPOSITORY-46EBA5?style=for-the-badge&logo=github&logoColor=07101A" alt="Open repository"/>
</a>

A full-stack project combining backend development, frontend components, database integration and deployment.

**Focus**

`Python` · `Django` · `React` · `PostgreSQL` · `Gunicorn`

---

## `04` / research

### Privacy-Preserving Machine Learning

My research explores the practical trade-offs between **privacy, model utility and computation cost** when machine learning is performed using encrypted data.

Areas I work around:

```text
Homomorphic Encryption
        ↓
Encrypted computation
        ↓
Privacy-preserving ML
        ↓
Healthcare / sensitive-data workloads
        ↓
Practical engineering + performance
