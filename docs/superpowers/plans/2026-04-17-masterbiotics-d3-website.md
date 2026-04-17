# MASTERBiotics D3.js 互動網站 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build 6 self-contained interactive HTML pages (3 B2C + 3 B2B) showcasing MASTERBiotics™ spore-forming probiotics, using d3.js for data visualization.

**Architecture:** Each HTML file is fully self-contained (inline CSS + JS). d3.js v7 loaded via CDN. All data hardcoded in JS. IntersectionObserver drives scroll-triggered animations. Shared color palette: dark blue-purple (#1a1a3e) + gold (#d4a843).

**Tech Stack:** HTML5, CSS3, JavaScript ES6+, d3.js v7 (CDN)

**Spec:** `docs/superpowers/specs/2026-04-17-masterbiotics-d3-website-design.md`

---

## Shared Patterns Reference

All 6 files use these same patterns. Each task includes the complete code — this section exists for quick reference only.

### Color Palette
```javascript
const COLORS = {
  bgDark: '#1a1a3e',
  bgPurple: '#2d1b69',
  gold: '#d4a843',
  white: '#ffffff',
  lightGray: '#ccccdd',
  purple: '#6b5ce7',
  blue: '#3a86c8',
  green: '#4caf50',
  orange: '#ff9800',
  gradientStart: '#1a1a3e',
  gradientEnd: '#2d1b69'
};
```

### Common CSS Reset
```css
* { margin: 0; padding: 0; box-sizing: border-box; }
html { scroll-behavior: smooth; }
body {
  font-family: -apple-system, 'Microsoft JhengHei', 'PingFang TC', system-ui, sans-serif;
  background: #1a1a3e;
  color: #ffffff;
  overflow-x: hidden;
}
```

### IntersectionObserver Pattern
```javascript
function setupScrollAnimations() {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('visible');
        if (entry.target.dataset.chart) {
          renderChart(entry.target.dataset.chart, entry.target);
        }
        observer.unobserve(entry.target);
      }
    });
  }, { threshold: 0.2 });
  document.querySelectorAll('.animate-on-scroll').forEach(el => observer.observe(el));
}
```

### Animated Counter Pattern
```javascript
function animateCounter(element, target, duration = 2000, suffix = '') {
  let start = 0;
  const startTime = performance.now();
  function update(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    const eased = 1 - Math.pow(1 - progress, 3);
    const current = Math.floor(eased * target);
    element.textContent = current.toLocaleString() + suffix;
    if (progress < 1) requestAnimationFrame(update);
  }
  requestAnimationFrame(update);
}
```

### D3 Bar Chart Factory
```javascript
function createBarChart(container, data, {
  width = 500, height = 300, margin = {top: 30, right: 20, bottom: 60, left: 60},
  xKey = 'group', yKey = 'value', color = '#d4a843',
  yLabel = '', xRotate = false, tooltip = true
} = {}) {
  const svg = d3.select(container).append('svg')
    .attr('viewBox', `0 0 ${width} ${height}`)
    .attr('preserveAspectRatio', 'xMidYMid meet')
    .style('width', '100%');
  const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`);
  const innerW = width - margin.left - margin.right;
  const innerH = height - margin.top - margin.bottom;
  const x = d3.scaleBand().domain(data.map(d => d[xKey])).range([0, innerW]).padding(0.3);
  const y = d3.scaleLinear().domain([0, d3.max(data, d => d[yKey]) * 1.1]).range([innerH, 0]);
  // Axes
  const xAxis = g.append('g').attr('transform', `translate(0,${innerH})`).call(d3.axisBottom(x));
  xAxis.selectAll('text').style('fill', '#ccccdd').style('font-size', '11px');
  if (xRotate) xAxis.selectAll('text').attr('transform', 'rotate(-30)').style('text-anchor', 'end');
  g.append('g').call(d3.axisLeft(y).ticks(5)).selectAll('text').style('fill', '#ccccdd');
  xAxis.select('.domain').style('stroke', '#555');
  g.selectAll('.tick line').style('stroke', '#333');
  if (yLabel) g.append('text').attr('transform', 'rotate(-90)').attr('y', -45).attr('x', -innerH/2)
    .attr('text-anchor', 'middle').style('fill', '#ccccdd').style('font-size', '12px').text(yLabel);
  // Bars with animation
  const bars = g.selectAll('.bar').data(data).enter().append('rect')
    .attr('class', 'bar')
    .attr('x', d => x(d[xKey]))
    .attr('width', x.bandwidth())
    .attr('y', innerH)
    .attr('height', 0)
    .attr('fill', (d, i) => typeof color === 'function' ? color(d, i) : color)
    .attr('rx', 3);
  bars.transition().duration(800).delay((d, i) => i * 100)
    .attr('y', d => y(d[yKey]))
    .attr('height', d => innerH - y(d[yKey]));
  // Tooltip
  if (tooltip) {
    const tip = d3.select(container).append('div')
      .style('position', 'absolute').style('background', 'rgba(0,0,0,0.8)')
      .style('color', '#fff').style('padding', '6px 10px').style('border-radius', '4px')
      .style('font-size', '12px').style('pointer-events', 'none').style('opacity', 0);
    bars.on('mouseover', function(event, d) {
      tip.style('opacity', 1).html(`${d[xKey]}: ${d[yKey]}`);
      d3.select(this).style('opacity', 0.8);
    }).on('mousemove', function(event) {
      tip.style('left', (event.offsetX + 10) + 'px').style('top', (event.offsetY - 30) + 'px');
    }).on('mouseout', function() {
      tip.style('opacity', 0);
      d3.select(this).style('opacity', 1);
    });
  }
  return { svg, g, x, y, bars };
}
```

### D3 Line Chart Factory
```javascript
function createLineChart(container, datasets, {
  width = 500, height = 300, margin = {top: 30, right: 100, bottom: 50, left: 60},
  xKey = 'time', yDomain = [50, 110], colors = ['#d4a843', '#6b5ce7'],
  labels = [], yLabel = ''
} = {}) {
  const svg = d3.select(container).append('svg')
    .attr('viewBox', `0 0 ${width} ${height}`)
    .attr('preserveAspectRatio', 'xMidYMid meet')
    .style('width', '100%');
  const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`);
  const innerW = width - margin.left - margin.right;
  const innerH = height - margin.top - margin.bottom;
  const allData = datasets[0];
  const x = d3.scalePoint().domain(allData.map(d => d[xKey])).range([0, innerW]);
  const y = d3.scaleLinear().domain(yDomain).range([innerH, 0]);
  g.append('g').attr('transform', `translate(0,${innerH})`).call(d3.axisBottom(x))
    .selectAll('text').style('fill', '#ccccdd');
  g.append('g').call(d3.axisLeft(y).ticks(5)).selectAll('text').style('fill', '#ccccdd');
  datasets.forEach((data, i) => {
    const line = d3.line().x(d => x(d[xKey])).y(d => y(d.value)).curve(d3.curveMonotoneX);
    const path = g.append('path').datum(data).attr('d', line)
      .attr('fill', 'none').attr('stroke', colors[i]).attr('stroke-width', 2.5);
    const totalLength = path.node().getTotalLength();
    path.attr('stroke-dasharray', totalLength).attr('stroke-dashoffset', totalLength)
      .transition().duration(1500).attr('stroke-dashoffset', 0);
    g.selectAll(`.dot-${i}`).data(data).enter().append('circle')
      .attr('cx', d => x(d[xKey])).attr('cy', d => y(d.value)).attr('r', 4)
      .attr('fill', colors[i]).style('opacity', 0)
      .transition().delay(1500).duration(300).style('opacity', 1);
    if (labels[i]) {
      g.append('text').attr('x', innerW + 10).attr('y', y(data[data.length-1].value))
        .style('fill', colors[i]).style('font-size', '12px').text(labels[i]);
    }
  });
}
```

### D3 Stacked Bar Chart Factory
```javascript
function createStackedBarChart(container, data, {
  width = 500, height = 300, margin = {top: 30, right: 20, bottom: 50, left: 50},
  keys = ['constipation', 'diarrhea', 'healthy'],
  colors = ['#ff9800', '#4caf50', '#3a86c8'],
  labels = ['便秘', '腹瀉', '健康'],
  xKey = 'day'
} = {}) {
  const svg = d3.select(container).append('svg')
    .attr('viewBox', `0 0 ${width} ${height}`)
    .attr('preserveAspectRatio', 'xMidYMid meet')
    .style('width', '100%');
  const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`);
  const innerW = width - margin.left - margin.right;
  const innerH = height - margin.top - margin.bottom;
  const x = d3.scaleBand().domain(data.map(d => d[xKey])).range([0, innerW]).padding(0.3);
  const y = d3.scaleLinear().domain([0, 100]).range([innerH, 0]);
  const stack = d3.stack().keys(keys)(data);
  const color = d3.scaleOrdinal().domain(keys).range(colors);
  g.append('g').attr('transform', `translate(0,${innerH})`).call(d3.axisBottom(x))
    .selectAll('text').style('fill', '#ccccdd');
  g.append('g').call(d3.axisLeft(y).tickFormat(d => d + '%')).selectAll('text').style('fill', '#ccccdd');
  g.selectAll('.serie').data(stack).enter().append('g')
    .attr('fill', d => color(d.key))
    .selectAll('rect').data(d => d).enter().append('rect')
    .attr('x', d => x(d.data[xKey])).attr('width', x.bandwidth())
    .attr('y', innerH).attr('height', 0)
    .transition().duration(800).delay((d, i) => i * 80)
    .attr('y', d => y(d[1])).attr('height', d => y(d[0]) - y(d[1]));
  // Legend
  const legend = svg.append('g').attr('transform', `translate(${margin.left}, ${height - 15})`);
  labels.forEach((label, i) => {
    const lg = legend.append('g').attr('transform', `translate(${i * 80}, 0)`);
    lg.append('rect').attr('width', 12).attr('height', 12).attr('fill', colors[i]).attr('rx', 2);
    lg.append('text').attr('x', 16).attr('y', 10).style('fill', '#ccccdd').style('font-size', '11px').text(label);
  });
}
```

### D3 Grouped Bar Chart Factory
```javascript
function createGroupedBarChart(container, data, {
  width = 500, height = 300, margin = {top: 30, right: 20, bottom: 60, left: 60},
  xKey = 'strain', groupKeys = ['control', 'treatment'],
  colors = ['#6b5ce7', '#3a86c8'],
  groupLabels = ['對照組', '處理組'],
  yLabel = ''
} = {}) {
  const svg = d3.select(container).append('svg')
    .attr('viewBox', `0 0 ${width} ${height}`)
    .attr('preserveAspectRatio', 'xMidYMid meet')
    .style('width', '100%');
  const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`);
  const innerW = width - margin.left - margin.right;
  const innerH = height - margin.top - margin.bottom;
  const x0 = d3.scaleBand().domain(data.map(d => d[xKey])).range([0, innerW]).padding(0.2);
  const x1 = d3.scaleBand().domain(groupKeys).range([0, x0.bandwidth()]).padding(0.05);
  const maxVal = d3.max(data, d => d3.max(groupKeys, k => d[k]));
  const y = d3.scaleLinear().domain([0, maxVal * 1.15]).range([innerH, 0]);
  g.append('g').attr('transform', `translate(0,${innerH})`).call(d3.axisBottom(x0))
    .selectAll('text').style('fill', '#ccccdd').style('font-size', '10px');
  g.append('g').call(d3.axisLeft(y).ticks(5)).selectAll('text').style('fill', '#ccccdd');
  if (yLabel) g.append('text').attr('transform', 'rotate(-90)').attr('y', -45).attr('x', -innerH/2)
    .attr('text-anchor', 'middle').style('fill', '#ccccdd').style('font-size', '12px').text(yLabel);
  const groups = g.selectAll('.group').data(data).enter().append('g')
    .attr('transform', d => `translate(${x0(d[xKey])},0)`);
  groups.selectAll('rect').data(d => groupKeys.map(k => ({key: k, value: d[k]}))).enter()
    .append('rect')
    .attr('x', d => x1(d.key)).attr('width', x1.bandwidth())
    .attr('y', innerH).attr('height', 0).attr('fill', d => colors[groupKeys.indexOf(d.key)]).attr('rx', 2)
    .transition().duration(800).delay((d, i) => i * 100)
    .attr('y', d => y(d.value)).attr('height', d => innerH - y(d.value));
  // Legend
  const legend = svg.append('g').attr('transform', `translate(${width - margin.right - 160}, ${margin.top})`);
  groupLabels.forEach((label, i) => {
    const lg = legend.append('g').attr('transform', `translate(0, ${i * 20})`);
    lg.append('rect').attr('width', 12).attr('height', 12).attr('fill', colors[i]).attr('rx', 2);
    lg.append('text').attr('x', 16).attr('y', 10).style('fill', '#ccccdd').style('font-size', '11px').text(label);
  });
}
```

---

## Task 1: b2c-a.html — B2C Scrollytelling

**Files:**
- Create: `b2c-a.html`

- [ ] **Step 1: Create complete b2c-a.html**

Create the file with all sections. Structure:

```
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>日本億菌大師 MASTERBiotics™ — 芽孢益生菌的革命</title>
  <script src="https://d3js.org/d3.v7.min.js"></script>
  <style>/* all CSS inline */</style>
</head>
<body>
  <!-- Section 1: Hero -->
  <!-- Section 2: 市場痛點 -->
  <!-- Section 3: 芽孢解方 -->
  <!-- Section 4: 五大功效 -->
  <!-- Section 5: 臨床實證 -->
  <!-- Section 6: 產品形式 + CTA -->
  <script>/* all JS inline */</script>
</body>
</html>
```

**CSS requirements:**
- Full-viewport sections (`min-height: 100vh`)
- `.animate-on-scroll` class: initial `opacity: 0; transform: translateY(40px); transition: all 0.8s ease`
- `.animate-on-scroll.visible`: `opacity: 1; transform: translateY(0)`
- Gradient backgrounds per section alternating `#1a1a3e` and `#2d1b69`
- Gold `#d4a843` for headings and accent text
- Responsive: sections stack vertically on mobile, charts scale via viewBox

**Section 1 — Hero:**
- Full-screen dark gradient background
- d3.js canvas particle animation: ~50 floating ellipses (bacteria shapes) slowly drifting
- Center: 「MASTER Biotics™」title, 「日本億菌大師」subtitle
- Below: 「500億凝結芽孢桿菌 + 2000億納豆芽孢桿菌」
- Scroll-down arrow indicator at bottom

```javascript
// Particle animation
function initParticles() {
  const canvas = document.getElementById('particle-canvas');
  const ctx = canvas.getContext('2d');
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
  const particles = Array.from({length: 50}, () => ({
    x: Math.random() * canvas.width,
    y: Math.random() * canvas.height,
    rx: Math.random() * 15 + 5,
    ry: Math.random() * 6 + 3,
    speed: Math.random() * 0.5 + 0.1,
    angle: Math.random() * Math.PI * 2,
    opacity: Math.random() * 0.3 + 0.1
  }));
  function animate() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    particles.forEach(p => {
      p.x += Math.cos(p.angle) * p.speed;
      p.y += Math.sin(p.angle) * p.speed;
      if (p.x < -20) p.x = canvas.width + 20;
      if (p.x > canvas.width + 20) p.x = -20;
      if (p.y < -20) p.y = canvas.height + 20;
      if (p.y > canvas.height + 20) p.y = -20;
      ctx.save();
      ctx.globalAlpha = p.opacity;
      ctx.fillStyle = '#6b5ce7';
      ctx.beginPath();
      ctx.ellipse(p.x, p.y, p.rx, p.ry, p.angle, 0, Math.PI * 2);
      ctx.fill();
      ctx.restore();
    });
    requestAnimationFrame(animate);
  }
  animate();
}
```

**Section 2 — 市場痛點 (3 cards slide in):**
```html
<section class="section" style="background: linear-gradient(135deg, #2d1b69, #1a1a3e)">
  <h2 class="section-title animate-on-scroll">益生菌市場的痛點</h2>
  <div class="pain-cards">
    <div class="pain-card animate-on-scroll" style="transition-delay: 0.1s">
      <div class="pain-icon">🌡️</div>
      <h3>活菌數快速下降</h3>
      <p>未冷藏狀態下，傳統益生菌活菌數快速流失</p>
    </div>
    <div class="pain-card animate-on-scroll" style="transition-delay: 0.3s">
      <div class="pain-icon">❄️</div>
      <h3>加工儲存嚴苛</h3>
      <p>需保持25°C以下，條件嚴苛</p>
    </div>
    <div class="pain-card animate-on-scroll" style="transition-delay: 0.5s">
      <div class="pain-icon">🚚</div>
      <h3>冷鏈成本高昂</h3>
      <p>運輸需全程冷鏈，成本大幅增加</p>
    </div>
  </div>
</section>
```

**Section 3 — 芽孢解方 (3 advantages light up):**
- Big text 「芽孢益生菌」fades in
- 3 advantage items appear with icons: 活菌數高且穩定 / 環境耐受性強 / 標示經得起考驗

**Section 4 — 五大功效 (radial layout):**
- d3.js radial/circular layout with center node「五大功效」
- 5 nodes around it, animate outward on scroll
- Click each node → expand panel with description

```javascript
function renderRadialBenefits(container) {
  const benefits = [
    { name: '腸道健康', desc: '耐胃酸膽鹽，腸道定殖力強', angle: -90 },
    { name: '免疫優化', desc: '抗病毒、抗過敏、抗發炎三重功效', angle: -18 },
    { name: '體重管理', desc: '激活GLP-1瘦瘦素', angle: 54 },
    { name: '好心情', desc: '腦腸軸促進多巴胺分泌', angle: 126 },
    { name: '抗氧化', desc: 'SOD + Catalase長效抗氧化', angle: 198 }
  ];
  const svg = d3.select(container).append('svg')
    .attr('viewBox', '0 0 600 600').style('width', '100%').style('max-width', '600px');
  const g = svg.append('g').attr('transform', 'translate(300,300)');
  // Center
  g.append('circle').attr('r', 60).attr('fill', '#d4a843');
  g.append('text').attr('text-anchor', 'middle').attr('dy', '0.35em')
    .style('fill', '#1a1a3e').style('font-weight', 'bold').style('font-size', '16px').text('五大功效');
  const radius = 180;
  benefits.forEach((b, i) => {
    const rad = b.angle * Math.PI / 180;
    const cx = Math.cos(rad) * radius;
    const cy = Math.sin(rad) * radius;
    const node = g.append('g').attr('class', 'benefit-node')
      .attr('transform', `translate(${cx},${cy})`).style('cursor', 'pointer');
    node.append('circle').attr('r', 0).attr('fill', '#6b5ce7')
      .transition().delay(i * 200).duration(600).attr('r', 50);
    node.append('text').attr('text-anchor', 'middle').attr('dy', '0.35em')
      .style('fill', '#fff').style('font-size', '13px').style('opacity', 0)
      .text(b.name).transition().delay(i * 200 + 400).duration(300).style('opacity', 1);
    // Connection line
    g.append('line').attr('x1', 0).attr('y1', 0).attr('x2', 0).attr('y2', 0)
      .attr('stroke', '#6b5ce7').attr('stroke-width', 1).attr('opacity', 0.4)
      .transition().delay(i * 200).duration(600)
      .attr('x2', cx).attr('y2', cy);
    // Click to expand
    node.on('click', () => {
      document.getElementById('benefit-detail').innerHTML = `<h3>${b.name}</h3><p>${b.desc}</p>`;
      document.getElementById('benefit-detail').classList.add('visible');
    });
  });
}
```

**Section 5 — 臨床實證:**
- Left side: d3.js stacked bar chart — Bristol Stool 排便改善 (use `createStackedBarChart` with whey B.coagulans data)
- Right side: animated counter — 「蛋白質吸收提升 200%+」
- Data from spec: `bristolWhey.bCoagulans` and absorption improvement numbers

**Section 6 — 產品形式 + CTA:**
- 5 product form icons in a grid (粉劑飲品、口服粉包、膠囊、錠劑、軟糖/果凍)
- Bottom CTA: 「了解更多」button

- [ ] **Step 2: Verify in browser**

Run: `open b2c-a.html` (macOS) and check:
1. Particle animation renders on hero
2. Scroll triggers section animations
3. Radial benefits chart expands on scroll
4. Bristol stacked bar chart animates
5. Counter animates to 200%+
6. Responsive on window resize

- [ ] **Step 3: Commit**

```bash
git add b2c-a.html
git commit -m "feat: add B2C scrollytelling page (b2c-a.html)"
```

---

## Task 2: b2c-b.html — B2C Dashboard

**Files:**
- Create: `b2c-b.html`

- [ ] **Step 1: Create complete b2c-b.html**

Structure:
```
<!DOCTYPE html> ...
<body>
  <!-- Hero Banner -->
  <!-- Dashboard Grid -->
  <!--   Panel 1: 功效卡片 -->
  <!--   Panel 2: 臨床數據（可切換蛋白質類型） -->
  <!--   Panel 3: 排便改善 -->
  <!--   Panel 4: 產品形式 -->
</body>
```

**CSS requirements:**
- Hero banner: `height: 40vh`, gradient background, centered brand + numbers
- Dashboard grid: `display: grid; grid-template-columns: repeat(2, 1fr); gap: 24px; padding: 40px`
- Panel cards: `background: rgba(45, 27, 105, 0.6); border-radius: 16px; padding: 30px; backdrop-filter: blur(10px)`
- Mobile: `grid-template-columns: 1fr`

**Hero Banner:**
```html
<header class="hero-banner">
  <h1>MASTER<span class="tm">™</span> Biotics</h1>
  <h2>日本億菌大師</h2>
  <div class="hero-numbers">
    <div class="hero-stat">
      <span class="stat-number" data-target="500">0</span><span class="stat-unit">億</span>
      <span class="stat-label">凝結芽孢桿菌</span>
    </div>
    <div class="hero-stat">
      <span class="stat-number" data-target="2000">0</span><span class="stat-unit">億</span>
      <span class="stat-label">納豆芽孢桿菌</span>
    </div>
  </div>
</header>
```

**Panel 1 — 功效卡片 (flip on hover):**
```html
<div class="panel">
  <h3 class="panel-title">五大健康功效</h3>
  <div class="benefit-grid">
    <!-- 5 cards, each with front (icon + name) and back (description) -->
    <div class="flip-card">
      <div class="flip-inner">
        <div class="flip-front"><div class="benefit-icon">🦠</div><span>腸道健康</span></div>
        <div class="flip-back"><p>耐胃酸膽鹽考驗，腸道定殖力強，經3小時胃酸及膽鹽處理後菌數無明顯下降</p></div>
      </div>
    </div>
    <!-- repeat for: 免疫優化🛡️, 體重管理⚖️, 好心情😊, 抗氧化✨ -->
  </div>
</div>
```

CSS for flip cards:
```css
.flip-card { perspective: 1000px; height: 160px; cursor: pointer; }
.flip-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
.flip-card:hover .flip-inner { transform: rotateY(180deg); }
.flip-front, .flip-back { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; border-radius: 12px; display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 16px; }
.flip-front { background: rgba(107, 92, 231, 0.3); }
.flip-back { background: rgba(212, 168, 67, 0.2); transform: rotateY(180deg); }
```

**Panel 2 — 臨床數據 (dropdown switcher):**
- Dropdown to select protein type: 乳清蛋白 / 碗豆蛋白 / 膠原蛋白
- d3.js grouped bar chart showing TN (總氮) and FAN (游離胺基酸氮) for control vs B.coagulans vs B.subtilis
- Data from spec: `tnFanWhey`, `tnFanPea`, `tnFanCollagen`

```javascript
function renderTnFanChart(proteinType) {
  const dataMap = { whey: tnFanWhey, pea: tnFanPea, collagen: tnFanCollagen };
  const d = dataMap[proteinType];
  const chartData = [
    { group: '無益生菌', tn: d.control.tn, fan: d.control.fan },
    { group: '凝結芽孢桿菌', tn: d.bCoagulans.tn, fan: d.bCoagulans.fan },
    { group: '納豆芽孢桿菌', tn: d.bSubtilis.tn, fan: d.bSubtilis.fan }
  ];
  const container = document.getElementById('tn-fan-chart');
  container.innerHTML = '';
  createGroupedBarChart(container, chartData, {
    xKey: 'group', groupKeys: ['tn', 'fan'],
    colors: ['#d4a843', '#4caf50'],
    groupLabels: ['糞便總氮 TN (%)', '游離胺基酸氮 FAN (%)'],
    yLabel: '%'
  });
}
```

**Panel 3 — 排便改善 (toggle B.coagulans / B.subtilis):**
- Two toggle buttons
- d3.js stacked bar chart using `createStackedBarChart`
- Default shows B.coagulans whey data
- Toggle switches data and re-renders with transition

**Panel 4 — 產品形式:**
- Simple icon grid, 5 items (粉劑飲品、口服粉包、膠囊、錠劑、軟糖/果凍)
- CSS-only hover effects

- [ ] **Step 2: Verify in browser**

Run: `open b2c-b.html` and check:
1. Hero counters animate on load
2. Flip cards work on hover
3. Dropdown switches chart data
4. Toggle buttons switch Bristol chart
5. Grid responsive on mobile

- [ ] **Step 3: Commit**

```bash
git add b2c-b.html
git commit -m "feat: add B2C dashboard page (b2c-b.html)"
```

---

## Task 3: b2c-c.html — B2C Product Showcase

**Files:**
- Create: `b2c-c.html`

- [ ] **Step 1: Create complete b2c-c.html**

Structure:
```
<body>
  <!-- Section 1: Full-screen Hero with animated counters -->
  <!-- Section 2: 消化道旅程 SVG animation -->
  <!-- Section 3: 功效輪播 -->
  <!-- Section 4: 數據亮點 -->
  <!-- Section 5: 產品陣列 -->
</body>
```

**Section 1 — Hero with animated counters:**
- Full-screen gradient background
- Brand name centered
- Two large animated counters: 500億 and 2000億 (use `animateCounter`)
- Parallax: slight background movement on scroll

```css
.hero-parallax {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(180deg, #1a1a3e 0%, #2d1b69 100%);
  position: relative;
  overflow: hidden;
}
.counter-group { display: flex; gap: 80px; margin-top: 40px; }
.counter-item { text-align: center; }
.counter-number { font-size: 72px; font-weight: 900; color: #d4a843; }
.counter-unit { font-size: 24px; color: #d4a843; }
.counter-label { font-size: 16px; color: #ccccdd; margin-top: 8px; }
```

**Section 2 — 消化道旅程 SVG animation:**
- SVG digestive tract path (口腔→胃→小腸→大腸)
- Animated dot travels along the path
- At each stage, a text box appears with description

```javascript
function renderDigestiveJourney(container) {
  const svg = d3.select(container).append('svg')
    .attr('viewBox', '0 0 800 600').style('width', '100%').style('max-width', '800px');
  // Simplified digestive tract path
  const pathData = 'M400,50 L400,120 C400,150 350,180 350,220 C350,280 450,280 450,220 L450,300 C450,350 300,370 300,420 C300,500 500,500 500,420 L500,550';
  const stages = [
    { pos: 0.05, label: '口腔', desc: '孢子經口腔進入人體', x: 500, y: 60 },
    { pos: 0.3, label: '胃部', desc: '在胃液中安全地吸收水份\n活化約3小時', x: 200, y: 200 },
    { pos: 0.6, label: '小腸', desc: '孢子開始在十二指腸發芽\n並在小腸上部增殖', x: 550, y: 380 },
    { pos: 0.9, label: '大腸', desc: '芽孢菌活化後下行至大腸\n並在下部發孢子', x: 250, y: 520 }
  ];
  // Draw path
  const path = svg.append('path').attr('d', pathData)
    .attr('fill', 'none').attr('stroke', '#6b5ce7').attr('stroke-width', 4).attr('opacity', 0.6);
  // Animated dot
  const dot = svg.append('circle').attr('r', 8).attr('fill', '#d4a843');
  const totalLength = path.node().getTotalLength();
  // Stage labels (hidden initially)
  stages.forEach((s, i) => {
    const g = svg.append('g').attr('class', `stage-${i}`).style('opacity', 0);
    g.append('rect').attr('x', s.x - 80).attr('y', s.y - 25).attr('width', 160).attr('height', 50)
      .attr('rx', 8).attr('fill', 'rgba(45,27,105,0.8)').attr('stroke', '#d4a843');
    g.append('text').attr('x', s.x).attr('y', s.y).attr('text-anchor', 'middle')
      .style('fill', '#d4a843').style('font-weight', 'bold').style('font-size', '14px').text(s.label);
    g.append('text').attr('x', s.x).attr('y', s.y + 18).attr('text-anchor', 'middle')
      .style('fill', '#ccccdd').style('font-size', '11px').text(s.desc.split('\n')[0]);
  });
  // Animate dot along path
  function animateJourney() {
    dot.transition().duration(6000).ease(d3.easeLinear)
      .attrTween('transform', function() {
        return function(t) {
          const p = path.node().getPointAtLength(t * totalLength);
          // Show stages as dot passes
          stages.forEach((s, i) => {
            if (t >= s.pos) svg.select(`.stage-${i}`).transition().duration(400).style('opacity', 1);
          });
          return `translate(${p.x},${p.y})`;
        };
      });
  }
  return { animateJourney };
}
```

**Section 3 — 功效輪播:**
- Full-width carousel with 5 slides (one per benefit)
- Auto-rotate every 4s, manual prev/next buttons
- Each slide: icon + title + description on gradient card

```javascript
function initCarousel() {
  const slides = document.querySelectorAll('.carousel-slide');
  let current = 0;
  function showSlide(i) {
    slides.forEach(s => s.classList.remove('active'));
    slides[i].classList.add('active');
    // Update dots
    document.querySelectorAll('.carousel-dot').forEach((d, idx) =>
      d.classList.toggle('active', idx === i));
  }
  document.getElementById('carousel-prev').onclick = () => {
    current = (current - 1 + slides.length) % slides.length;
    showSlide(current);
  };
  document.getElementById('carousel-next').onclick = () => {
    current = (current + 1) % slides.length;
    showSlide(current);
  };
  setInterval(() => { current = (current + 1) % slides.length; showSlide(current); }, 4000);
}
```

**Section 4 — 數據亮點:**
- 4 large stat blocks: 「改善70%」「吸收提升200%+」「100%存活率」「2500億總菌數」
- Each triggers `animateCounter` on scroll intersection

**Section 5 — 產品陣列:**
- Horizontal scroll container with 5 product cards
- Each card: icon + name + short description

- [ ] **Step 2: Verify in browser**

Run: `open b2c-c.html` and check:
1. Counter animation on hero
2. Digestive journey SVG animates dot along path
3. Carousel auto-rotates and manual buttons work
4. Stat counters trigger on scroll
5. Horizontal scroll on product section

- [ ] **Step 3: Commit**

```bash
git add b2c-c.html
git commit -m "feat: add B2C product showcase page (b2c-c.html)"
```

---

## Task 4: b2b-a.html — B2B Scrollytelling

**Files:**
- Create: `b2b-a.html`

- [ ] **Step 1: Create complete b2b-a.html**

Structure:
```
<body>
  <!-- Section 1: 市場洞察 (consumer survey bar chart) -->
  <!-- Section 2: 產品定位 (strain spec cards) -->
  <!-- Section 3: 科學驗證故事線 (6 stations) -->
  <!-- Section 4: 臨床試驗 (interactive panel) -->
  <!-- Section 5: 市場優勢 (comparison + formulas) -->
  <!-- Section 6: 關於 MCB -->
</body>
```

**All data constants (include at top of script):**

```javascript
// Consumer survey
const surveyData = [
  { factor: '活菌數', value: 70 },
  { factor: '功能性', value: 15 },
  { factor: '品牌', value: 10 }
];

// Acid-bile resistance (line chart)
const acidBileData = {
  bs: [
    { time: '0h', value: 100 }, { time: '1.5h', value: 101 },
    { time: '3h', value: 102 }, { time: '4.5h', value: 103 }, { time: '6h', value: 104 }
  ],
  bc: [
    { time: '0h', value: 100 }, { time: '1.5h', value: 100 },
    { time: '3h', value: 100 }, { time: '4.5h', value: 101 }, { time: '6h', value: 102 }
  ]
};

// Colonization
const colonizationData = [
  { strain: '凝結芽孢桿菌\nDSM35067', control: 5, withBrilliaNest: 18 },
  { strain: '納豆芽孢桿菌\nDSM35068', control: 7.5, withBrilliaNest: 21 },
  { strain: '嗜酸乳桿菌', control: 5, withBrilliaNest: 14 },
  { strain: '雷特式B菌', control: 6, withBrilliaNest: 17 }
];

// Immune markers
const immuneData = {
  ifnBeta: [
    { group: 'Control', value: 130 }, { group: 'BC', value: 140 },
    { group: 'BS', value: 135 }, { group: 'LA', value: 130 }, { group: 'BL', value: 125 }
  ],
  ifnGamma: [
    { group: 'Control', value: 100 }, { group: 'BC', value: 500 },
    { group: 'BS', value: 480 }, { group: 'LA', value: 450 }, { group: 'BL', value: 420 }
  ],
  tgfBeta: [
    { group: 'Control', value: 100 }, { group: 'Pathogen', value: 120 },
    { group: 'BC', value: 200 }, { group: 'BS', value: 190 },
    { group: 'LA', value: 160 }, { group: 'BL', value: 150 }
  ],
  il4: [
    { group: 'Control', value: 100 }, { group: 'Allergen', value: 210 },
    { group: 'BC', value: 120 }, { group: 'BS', value: 110 },
    { group: 'LA', value: 150 }, { group: 'BL', value: 140 }
  ],
  ifnIl4Ratio: [
    { group: 'Control', value: 100 }, { group: 'BC', value: 200 },
    { group: 'BS', value: 190 }, { group: 'LA', value: 100 }, { group: 'BL', value: 80 }
  ]
};

// GLP-1
const glp1Data = [
  { group: 'Control', value: 100 }, { group: 'BC-V', value: 480 },
  { group: 'BC-S', value: 450 }, { group: 'BS-V', value: 500 },
  { group: 'BS-S', value: 470 }, { group: 'LA', value: 250 }, { group: 'BL', value: 240 }
];

// Dopamine
const dopamineData = [
  { group: 'Control', value: 20 }, { group: 'BC-V', value: 130 },
  { group: 'BC-S', value: 130 }, { group: 'BC-L', value: 230 },
  { group: 'BS-V', value: 130 }, { group: 'BS-S', value: 130 }, { group: 'BS-L', value: 230 }
];

// Antioxidant enzymes
const catalaseData = [
  { strain: '億菌大師 DSM35067', value: 8000 },
  { strain: '它牌凝結芽孢菌S', value: 3500 },
  { strain: '它牌凝結芽孢菌B', value: 14000 },
  { strain: '億菌大師 DSM35068', value: 9500 },
  { strain: '它牌納豆芽孢菌H', value: 7000 },
  { strain: '它牌納豆芽孢菌S', value: 7000 }
];
const sodData = [
  { strain: '億菌大師 DSM35067', value: 800 },
  { strain: '它牌凝結芽孢菌S', value: 200 },
  { strain: '它牌凝結芽孢菌B', value: 350 },
  { strain: '億菌大師 DSM35068', value: 650 },
  { strain: '它牌納豆芽孢菌H', value: 550 },
  { strain: '它牌納豆芽孢菌S', value: 550 }
];

// Bristol stool data (all 9 groups)
const bristolData = {
  whey: {
    control: [
      { day: 'Baseline', constipation: 20, diarrhea: 20, healthy: 60 },
      { day: '1st', constipation: 20, diarrhea: 20, healthy: 60 },
      { day: '2nd', constipation: 15, diarrhea: 20, healthy: 65 },
      { day: '3rd', constipation: 15, diarrhea: 15, healthy: 70 },
      { day: '4th', constipation: 15, diarrhea: 15, healthy: 70 },
      { day: '5th', constipation: 15, diarrhea: 15, healthy: 70 }
    ],
    bCoagulans: [
      { day: 'Baseline', constipation: 25, diarrhea: 15, healthy: 60 },
      { day: '1st', constipation: 15, diarrhea: 15, healthy: 70 },
      { day: '2nd', constipation: 10, diarrhea: 10, healthy: 80 },
      { day: '3rd', constipation: 8, diarrhea: 7, healthy: 85 },
      { day: '4th', constipation: 7, diarrhea: 5, healthy: 88 },
      { day: '5th', constipation: 7, diarrhea: 5, healthy: 88 }
    ],
    bSubtilis: [
      { day: 'Baseline', constipation: 25, diarrhea: 15, healthy: 60 },
      { day: '1st', constipation: 15, diarrhea: 12, healthy: 73 },
      { day: '2nd', constipation: 10, diarrhea: 10, healthy: 80 },
      { day: '3rd', constipation: 8, diarrhea: 7, healthy: 85 },
      { day: '4th', constipation: 7, diarrhea: 5, healthy: 88 },
      { day: '5th', constipation: 7, diarrhea: 5, healthy: 88 }
    ]
  },
  pea: {
    control: [
      { day: 'Baseline', constipation: 20, diarrhea: 25, healthy: 55 },
      { day: '1st', constipation: 20, diarrhea: 25, healthy: 55 },
      { day: '2nd', constipation: 20, diarrhea: 20, healthy: 60 },
      { day: '3rd', constipation: 18, diarrhea: 20, healthy: 62 },
      { day: '4th', constipation: 18, diarrhea: 20, healthy: 62 },
      { day: '5th', constipation: 18, diarrhea: 20, healthy: 62 }
    ],
    bCoagulans: [
      { day: 'Baseline', constipation: 25, diarrhea: 20, healthy: 55 },
      { day: '1st', constipation: 15, diarrhea: 15, healthy: 70 },
      { day: '2nd', constipation: 10, diarrhea: 10, healthy: 80 },
      { day: '3rd', constipation: 10, diarrhea: 5, healthy: 85 },
      { day: '4th', constipation: 10, diarrhea: 2, healthy: 88 },
      { day: '5th', constipation: 10, diarrhea: 2, healthy: 88 }
    ],
    bSubtilis: [
      { day: 'Baseline', constipation: 25, diarrhea: 20, healthy: 55 },
      { day: '1st', constipation: 15, diarrhea: 12, healthy: 73 },
      { day: '2nd', constipation: 10, diarrhea: 8, healthy: 82 },
      { day: '3rd', constipation: 10, diarrhea: 5, healthy: 85 },
      { day: '4th', constipation: 10, diarrhea: 2, healthy: 88 },
      { day: '5th', constipation: 10, diarrhea: 2, healthy: 88 }
    ]
  },
  collagen: {
    control: [
      { day: 'Baseline', constipation: 15, diarrhea: 20, healthy: 65 },
      { day: '1st', constipation: 15, diarrhea: 20, healthy: 65 },
      { day: '2nd', constipation: 15, diarrhea: 18, healthy: 67 },
      { day: '3rd', constipation: 12, diarrhea: 18, healthy: 70 },
      { day: '4th', constipation: 12, diarrhea: 18, healthy: 70 },
      { day: '5th', constipation: 12, diarrhea: 18, healthy: 70 }
    ],
    bCoagulans: [
      { day: 'Baseline', constipation: 15, diarrhea: 20, healthy: 65 },
      { day: '1st', constipation: 12, diarrhea: 15, healthy: 73 },
      { day: '2nd', constipation: 10, diarrhea: 10, healthy: 80 },
      { day: '3rd', constipation: 10, diarrhea: 5, healthy: 85 },
      { day: '4th', constipation: 10, diarrhea: 2, healthy: 88 },
      { day: '5th', constipation: 10, diarrhea: 2, healthy: 88 }
    ],
    bSubtilis: [
      { day: 'Baseline', constipation: 15, diarrhea: 20, healthy: 65 },
      { day: '1st', constipation: 12, diarrhea: 15, healthy: 73 },
      { day: '2nd', constipation: 10, diarrhea: 10, healthy: 80 },
      { day: '3rd', constipation: 10, diarrhea: 5, healthy: 85 },
      { day: '4th', constipation: 10, diarrhea: 2, healthy: 88 },
      { day: '5th', constipation: 10, diarrhea: 2, healthy: 88 }
    ]
  }
};

// TN/FAN data
const tnFanData = {
  whey: {
    control: { tn: 10.23, fan: 0.65, ratio: 0.06 },
    bCoagulans: { tn: 5.98, fan: 1.20, ratio: 0.20 },
    bSubtilis: { tn: 5.91, fan: 1.28, ratio: 0.22 }
  },
  pea: {
    control: { tn: 11.01, fan: 0.59, ratio: 0.05 },
    bCoagulans: { tn: 6.19, fan: 1.22, ratio: 0.20 },
    bSubtilis: { tn: 6.24, fan: 1.38, ratio: 0.22 }
  },
  collagen: {
    control: { tn: 7.45, fan: 0.80, ratio: 0.10 },
    bCoagulans: { tn: 5.90, fan: 1.32, ratio: 0.22 },
    bSubtilis: { tn: 5.93, fan: 1.34, ratio: 0.23 }
  }
};

// Comparison table
const comparisonData = [
  { feature: '原料規格', masterBiotics: 'B.Coagulans 500億 / B.Subtilis 2000億', bCoagulans: '150億', bSubtilis: '1000億', lactoBifido: '100-200億' },
  { feature: '儲存條件', masterBiotics: '無需冷藏與冷鏈運輸', bCoagulans: '無需冷藏與冷鏈運輸', bSubtilis: '無需冷藏與冷鏈運輸', lactoBifido: '儲存溫度不能超過28°C' },
  { feature: '應用範圍', masterBiotics: '保健食品(含錠劑) 烘焙及糖果食品', bCoagulans: '健康補充劑(含錠劑) 烘焙及糖果食品', bSubtilis: '健康補充劑(含錠劑) 烘焙及糖果食品', lactoBifido: '膠囊和粉劑保健品' },
  { feature: '健康功能', masterBiotics: '腸道、免疫、體重管理和認知健康多重功能', bCoagulans: '視供應商而定', bSubtilis: '視供應商而定', lactoBifido: '視供應商而定' }
];

// Formula suggestions
const formulaData = [
  { name: '體重控制', ingredients: '羥基檸檬酸、生物類黃酮、EGCG、MCT油、白藜蘆醇、綠原酸', icon: '🏃' },
  { name: '提升免疫', ingredients: '多醣體、薑黃素、維生素C&D、母乳寡糖、蜂膠', icon: '🛡️' },
  { name: '蛋白增肌', ingredients: '乳清蛋白、素食蛋白、奶昔代餐、特膳食品', icon: '💪' },
  { name: '情緒穩定', ingredients: 'GABA、蘑菇多醣體、迷迭香酸、菸胺酸', icon: '😊' },
  { name: '促進消化', ingredients: '加州梅、纖維、油粉、益生元', icon: '🍎' },
  { name: '降低發炎', ingredients: '薑黃素、麩胱甘肽、維生素C、多酚', icon: '🔥' }
];
```

**Section 1 — 市場洞察:**
- d3.js bar chart of consumer survey data using `createBarChart(container, surveyData, { xKey: 'factor', yKey: 'value', color: '#d4a843' })`
- Accompanying text: 「消費者更重視活菌數」

**Section 2 — 產品定位:**
- Two side-by-side cards for DSM 35067 and DSM 35068
- Each card: strain name, count, DSMZ certification note

**Section 3 — 科學驗證 (6 stations, scroll-triggered):**
Each station is a full-viewport section with a d3.js chart:
- Station 1: `createLineChart(container, [acidBileData.bs, acidBileData.bc], { labels: ['納豆芽孢桿菌 DSM35068', '凝結芽孢桿菌 DSM35067'], yDomain: [50, 110] })`
- Station 2: `createGroupedBarChart(container, colonizationData, { xKey: 'strain', groupKeys: ['control', 'withBrilliaNest'], colors: ['#6b5ce7', '#3a86c8'], groupLabels: ['對照組', '添加燕窩酸組'], yLabel: '益生菌黏附力 (%)' })`
- Station 3: 4 sub-charts for immune data (IFN-β, IFN-γ, TGF-β, IL-4) in 2x2 grid, each using `createBarChart`
- Station 4: `createBarChart(container, glp1Data, { color: (d) => d.group.startsWith('B') ? '#3a86c8' : (d.group === 'Control' ? '#666' : '#6b5ce7'), yLabel: 'GLP-1 (pg/ml)' })`
- Station 5: `createBarChart(container, dopamineData, { color: (d) => d.group.includes('-L') ? '#d4a843' : '#3a86c8', yLabel: 'Dopamine (pg/ml)' })`
- Station 6: Two side-by-side charts using `createBarChart` for catalaseData and sodData, highlight MCB strains in gold

**Section 4 — 臨床試驗 (interactive panel):**
- Top: trial summary (150人、隨機雙盲、9組)
- Dropdown 1: protein type (乳清/碗豆/膠原)
- Dropdown 2: condition (對照/凝結芽孢桿菌/納豆芽孢桿菌)
- Bristol stool stacked bar chart updates on selection
- Below: TN/FAN grouped bar chart updates on protein type selection

**Section 5 — 市場優勢:**
- HTML table for comparison data with hover row highlight
- Formula grid: 6 cards in 3x2 layout

```html
<table class="comparison-table">
  <thead>
    <tr>
      <th></th>
      <th class="highlight">MASTER Biotics™</th>
      <th>B. coagulans</th>
      <th>B. subtilis</th>
      <th>乳酸菌/雙歧桿菌</th>
    </tr>
  </thead>
  <tbody>
    <!-- rows from comparisonData -->
  </tbody>
</table>
```

```css
.comparison-table { width: 100%; border-collapse: collapse; }
.comparison-table th, .comparison-table td { padding: 16px; text-align: left; border-bottom: 1px solid #333; }
.comparison-table th.highlight { color: #d4a843; }
.comparison-table tr:hover { background: rgba(107, 92, 231, 0.2); }
```

**Section 6 — 關於 MCB:**
- Company info: 1987年成立, 30+ years
- Certification logos as text badges: FSSC 22000, HACCP, ISO 22000, FDA, Halal
- Website: www.mingchyi.com

- [ ] **Step 2: Verify in browser**

Run: `open b2b-a.html` and check:
1. Consumer survey chart animates on scroll
2. All 6 experiment stations render charts correctly
3. Clinical trial dropdowns switch data
4. Comparison table hover works
5. All sections scroll-trigger properly

- [ ] **Step 3: Commit**

```bash
git add b2b-a.html
git commit -m "feat: add B2B scrollytelling page (b2b-a.html)"
```

---

## Task 5: b2b-b.html — B2B Dashboard

**Files:**
- Create: `b2b-b.html`

- [ ] **Step 1: Create complete b2b-b.html**

Structure:
```
<body>
  <div class="dashboard-layout">
    <nav class="side-nav"><!-- 5 tabs --></nav>
    <main class="content-panel"><!-- tab content --></main>
  </div>
</body>
```

**CSS for dashboard layout:**
```css
.dashboard-layout { display: flex; min-height: 100vh; }
.side-nav {
  width: 220px; background: #15132b; padding: 20px 0;
  display: flex; flex-direction: column; position: fixed; height: 100vh; z-index: 10;
}
.nav-item {
  padding: 16px 24px; color: #ccccdd; cursor: pointer;
  border-left: 3px solid transparent; transition: all 0.3s;
}
.nav-item:hover, .nav-item.active { color: #d4a843; background: rgba(107,92,231,0.15); border-left-color: #d4a843; }
.content-panel { margin-left: 220px; padding: 40px; flex: 1; }
.tab-content { display: none; }
.tab-content.active { display: block; }
@media (max-width: 768px) {
  .side-nav { width: 100%; height: auto; position: relative; flex-direction: row; overflow-x: auto; }
  .content-panel { margin-left: 0; }
  .nav-item { padding: 12px 16px; white-space: nowrap; border-left: none; border-bottom: 3px solid transparent; }
  .nav-item.active { border-bottom-color: #d4a843; }
}
```

**Tab switching JS:**
```javascript
function initTabs() {
  const navItems = document.querySelectorAll('.nav-item');
  const tabs = document.querySelectorAll('.tab-content');
  navItems.forEach((item, i) => {
    item.addEventListener('click', () => {
      navItems.forEach(n => n.classList.remove('active'));
      tabs.forEach(t => t.classList.remove('active'));
      item.classList.add('active');
      tabs[i].classList.add('active');
      // Render charts on first visit
      if (!tabs[i].dataset.rendered) {
        renderTabCharts(i);
        tabs[i].dataset.rendered = 'true';
      }
    });
  });
  // Default first tab
  navItems[0].click();
}
```

**Nav items:**
```html
<nav class="side-nav">
  <div class="nav-brand">
    <h2 style="color: #d4a843; padding: 20px 24px;">MASTER<br>Biotics™</h2>
  </div>
  <div class="nav-item active" data-tab="0">📋 菌株規格</div>
  <div class="nav-item" data-tab="1">🔬 實驗數據</div>
  <div class="nav-item" data-tab="2">📊 臨床試驗</div>
  <div class="nav-item" data-tab="3">🧪 配方建議</div>
  <div class="nav-item" data-tab="4">🏢 公司 + 比較</div>
</nav>
```

**Tab 1 — 菌株規格:**
- Two specification cards side-by-side
- Card content: strain name (gold), DSM number, count per gram, scientific notation, DSMZ certification

**Tab 2 — 實驗數據 (6 sub-tabs):**
- Sub-tab bar at top: 耐酸膽鹽 | 腸道定殖 | 免疫指標 | GLP-1 | 多巴胺 | 抗氧化
- Each sub-tab renders its d3.js chart
- Use the same chart factory functions and data from Task 4
- Sub-tab switching:

```javascript
function initSubTabs(parentId) {
  const subNavs = document.querySelectorAll(`#${parentId} .sub-tab`);
  const subPanels = document.querySelectorAll(`#${parentId} .sub-panel`);
  subNavs.forEach((nav, i) => {
    nav.addEventListener('click', () => {
      subNavs.forEach(n => n.classList.remove('active'));
      subPanels.forEach(p => p.classList.remove('active'));
      nav.classList.add('active');
      subPanels[i].classList.add('active');
      if (!subPanels[i].dataset.rendered) {
        renderSubChart(i, subPanels[i]);
        subPanels[i].dataset.rendered = 'true';
      }
    });
  });
  subNavs[0].click();
}
```

**Tab 3 — 臨床試驗:**
- Trial method summary at top
- Dropdown: protein type selector
- Dropdown: condition selector (control / B.coagulans / B.subtilis)
- Two chart areas: Bristol stacked bar + TN/FAN bar chart
- All Bristol and TN/FAN data included (same data constants as Task 4)

**Tab 4 — 配方建議:**
- 3x2 grid of formula cards
- Each card has icon, name, ingredient list
- Click to expand with animation

**Tab 5 — 公司 + 比較:**
- Comparison table (same HTML structure as Task 4)
- MCB company section: history, certifications
- Certification badges in a row

- [ ] **Step 2: Verify in browser**

Run: `open b2b-b.html` and check:
1. Side navigation switches tabs
2. Sub-tabs in experiment data work
3. Charts render correctly on first tab visit
4. Clinical trial dropdowns update charts
5. Mobile responsive (nav becomes horizontal)

- [ ] **Step 3: Commit**

```bash
git add b2b-b.html
git commit -m "feat: add B2B dashboard page (b2b-b.html)"
```

---

## Task 6: b2b-c.html — B2B Product Showcase

**Files:**
- Create: `b2b-c.html`

- [ ] **Step 1: Create complete b2b-c.html**

Structure:
```
<body>
  <!-- Section 1: Hero -->
  <!-- Section 2: 菌株卡片 (flip on hover) -->
  <!-- Section 3: 數據亮點 (animated counters) -->
  <!-- Section 4: 實驗圖表輪播 -->
  <!-- Section 5: 比較優勢 -->
  <!-- Section 6: 配方 grid -->
  <!-- Section 7: MCB 公司 -->
</body>
```

**Section 1 — Hero:**
- Full-screen gradient
- MASTER Biotics™ brand text
- MCB Biotechnology Nutritionals
- Tagline: 「For Food & Nutrition, we always do more than expected」
- Subtle particle background (same as b2c-a but fewer particles, slower)

**Section 2 — 菌株卡片 (flip cards):**
- Two large cards (50% width each)
- Front: strain name + DSM number + large count number
- Back: detailed specs (count per gram, scientific notation, DSMZ certification)
- CSS flip effect (same pattern as b2c-b but larger)

```html
<div class="strain-cards">
  <div class="strain-flip-card">
    <div class="strain-flip-inner">
      <div class="strain-front">
        <h3>凝結芽孢桿菌</h3>
        <p class="dsm-number">DSM 35067</p>
        <p class="strain-count">500<span class="count-unit">億/克</span></p>
      </div>
      <div class="strain-back">
        <h4>Bacillus coagulans</h4>
        <ul>
          <li>菌數：500億/克 (5×10¹⁰)</li>
          <li>另有 150億/克 (1.5×10¹⁰) 規格</li>
          <li>經德國DSMZ菌庫認證</li>
          <li>無需冷藏與冷鏈運輸</li>
        </ul>
      </div>
    </div>
  </div>
  <!-- repeat for DSM 35068 -->
</div>
```

**Section 3 — 數據亮點 (5 counters):**
- Horizontal row of 5 large stat blocks
- Each triggers `animateCounter` on scroll:
  - 500億 (凝結芽孢桿菌)
  - 2000億 (納豆芽孢桿菌)
  - 100% (胃酸膽鹽存活率)
  - 200%+ (蛋白質吸收提升)
  - 70% (排便改善)

**Section 4 — 實驗圖表輪播:**
- Carousel with prev/next arrows
- 6 slides, each with one d3.js chart + title + conclusion text
- Slide 1: Acid-bile line chart + 「經3小時胃酸及膽鹽處理後菌數無明顯下降」
- Slide 2: Colonization grouped bar + 「與BrilliaNest™結合後定殖率大幅提升」
- Slide 3: Immune 4-chart grid + 「抗病毒、抗過敏、抗發炎三重免疫功效」
- Slide 4: GLP-1 bar + 「芽孢型態激活GLP-1優於市售益生菌」
- Slide 5: Dopamine bar + 「菌株裂解物多巴胺刺激能力最強」
- Slide 6: Catalase + SOD dual chart + 「抗氧化酵素活性領先競品」

```javascript
function initChartCarousel() {
  const slides = document.querySelectorAll('.chart-slide');
  let current = 0;
  const chartRendered = new Set();

  function showSlide(i) {
    slides.forEach(s => { s.style.display = 'none'; s.classList.remove('active'); });
    slides[i].style.display = 'block';
    slides[i].classList.add('active');
    // Render chart on first show
    if (!chartRendered.has(i)) {
      renderCarouselChart(i, slides[i].querySelector('.chart-container'));
      chartRendered.add(i);
    }
    // Update indicators
    document.querySelectorAll('.carousel-indicator').forEach((ind, idx) =>
      ind.classList.toggle('active', idx === i));
    document.getElementById('carousel-counter').textContent = `${i + 1} / ${slides.length}`;
  }

  document.getElementById('chart-prev').onclick = () => {
    current = (current - 1 + slides.length) % slides.length;
    showSlide(current);
  };
  document.getElementById('chart-next').onclick = () => {
    current = (current + 1) % slides.length;
    showSlide(current);
  };
  showSlide(0);
}

function renderCarouselChart(index, container) {
  switch (index) {
    case 0:
      createLineChart(container, [acidBileData.bs, acidBileData.bc], {
        labels: ['納豆芽孢桿菌 DSM35068', '凝結芽孢桿菌 DSM35067'],
        yDomain: [50, 110], yLabel: '存活率 (%)'
      });
      break;
    case 1:
      createGroupedBarChart(container, colonizationData, {
        xKey: 'strain', groupKeys: ['control', 'withBrilliaNest'],
        colors: ['#6b5ce7', '#3a86c8'],
        groupLabels: ['對照組', '添加燕窩酸組'], yLabel: '益生菌黏附力 (%)'
      });
      break;
    case 2:
      // 2x2 grid of immune charts
      const immuneGrid = document.createElement('div');
      immuneGrid.style.cssText = 'display:grid;grid-template-columns:1fr 1fr;gap:16px;';
      container.appendChild(immuneGrid);
      ['ifnBeta', 'ifnGamma', 'tgfBeta', 'il4'].forEach(key => {
        const div = document.createElement('div');
        immuneGrid.appendChild(div);
        const titles = { ifnBeta: 'IFN-β', ifnGamma: 'IFN-γ', tgfBeta: 'TGF-β', il4: 'IL-4' };
        const titleEl = document.createElement('h4');
        titleEl.textContent = titles[key];
        titleEl.style.cssText = 'color:#d4a843;text-align:center;margin-bottom:8px;';
        div.appendChild(titleEl);
        createBarChart(div, immuneData[key], {
          width: 300, height: 200, xKey: 'group', yKey: 'value',
          color: (d) => ['BC', 'BS'].includes(d.group) ? '#3a86c8' : '#6b5ce7',
          yLabel: 'pg/ml'
        });
      });
      break;
    case 3:
      createBarChart(container, glp1Data, {
        color: (d) => d.group.startsWith('B') ? '#3a86c8' : (d.group === 'Control' ? '#666' : '#6b5ce7'),
        yLabel: 'GLP-1 (pg/ml)'
      });
      break;
    case 4:
      createBarChart(container, dopamineData, {
        color: (d) => d.group.includes('-L') ? '#d4a843' : '#3a86c8',
        yLabel: 'Dopamine (pg/ml)'
      });
      break;
    case 5:
      const dualGrid = document.createElement('div');
      dualGrid.style.cssText = 'display:grid;grid-template-columns:1fr 1fr;gap:24px;';
      container.appendChild(dualGrid);
      const catDiv = document.createElement('div');
      const sodDiv = document.createElement('div');
      dualGrid.appendChild(catDiv);
      dualGrid.appendChild(sodDiv);
      const catTitle = document.createElement('h4');
      catTitle.textContent = 'Catalase 活性 (U/mL)';
      catTitle.style.cssText = 'color:#d4a843;text-align:center;margin-bottom:8px;';
      catDiv.appendChild(catTitle);
      createBarChart(catDiv, catalaseData, {
        xKey: 'strain', yKey: 'value', xRotate: true,
        color: (d) => d.strain.includes('億菌大師') ? '#d4a843' : '#6b5ce7',
        yLabel: 'U/mL'
      });
      const sodTitle = document.createElement('h4');
      sodTitle.textContent = 'SOD 活性 (U/mL)';
      sodTitle.style.cssText = 'color:#d4a843;text-align:center;margin-bottom:8px;';
      sodDiv.appendChild(sodTitle);
      createBarChart(sodDiv, sodData, {
        xKey: 'strain', yKey: 'value', xRotate: true,
        color: (d) => d.strain.includes('億菌大師') ? '#d4a843' : '#6b5ce7',
        yLabel: 'U/mL'
      });
      break;
  }
}
```

**Section 5 — 比較優勢:**
- Comparison table (same structure as Task 4)
- Hover row highlight, MCB column highlighted in gold

**Section 6 — 配方 grid:**
- 3x2 grid of formula cards (same data as Task 4)
- Hover: scale up slightly + glow border

**Section 7 — MCB 公司:**
- Company history: 「1987年成立於台灣，超過30年歷史」
- Certification badges row
- 「www.mingchyi.com」

- [ ] **Step 2: Verify in browser**

Run: `open b2b-c.html` and check:
1. Hero renders with brand text
2. Strain flip cards work on hover
3. Counter animations trigger on scroll
4. Chart carousel prev/next works, charts render
5. Comparison table hover highlight
6. All sections visible and properly styled

- [ ] **Step 3: Commit**

```bash
git add b2b-c.html
git commit -m "feat: add B2B product showcase page (b2b-c.html)"
```

---

## Task 7: Final Review and Index Page

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create index.html as navigation hub**

Simple navigation page linking to all 6 HTML files:

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>MASTERBiotics™ 互動網站系列</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: -apple-system, 'Microsoft JhengHei', 'PingFang TC', system-ui, sans-serif;
      background: #1a1a3e; color: #ffffff; min-height: 100vh;
      display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 40px;
    }
    h1 { color: #d4a843; margin-bottom: 8px; font-size: 36px; }
    .subtitle { color: #ccccdd; margin-bottom: 48px; font-size: 18px; }
    .grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 24px; max-width: 900px; width: 100%; }
    .section-label { grid-column: 1 / -1; color: #d4a843; font-size: 20px; font-weight: bold; margin-top: 16px; border-bottom: 1px solid #333; padding-bottom: 8px; }
    .card {
      background: rgba(45, 27, 105, 0.6); border-radius: 12px; padding: 24px;
      text-decoration: none; color: #fff; transition: all 0.3s;
      border: 1px solid rgba(107, 92, 231, 0.3);
    }
    .card:hover { transform: translateY(-4px); border-color: #d4a843; background: rgba(45, 27, 105, 0.8); }
    .card h3 { color: #d4a843; margin-bottom: 8px; }
    .card p { color: #ccccdd; font-size: 14px; line-height: 1.5; }
    .style-tag { display: inline-block; background: rgba(107,92,231,0.3); padding: 2px 8px; border-radius: 4px; font-size: 12px; margin-bottom: 8px; }
    @media (max-width: 768px) { .grid { grid-template-columns: 1fr; } }
  </style>
</head>
<body>
  <h1>MASTER Biotics™</h1>
  <p class="subtitle">日本億菌大師 — 互動網站系列</p>
  <div class="grid">
    <div class="section-label">B2C 消費者版</div>
    <a href="b2c-a.html" class="card">
      <span class="style-tag">Scrollytelling</span>
      <h3>滾動敘事版</h3>
      <p>全屏滾動動畫，一步步帶入芽孢益生菌的世界</p>
    </a>
    <a href="b2c-b.html" class="card">
      <span class="style-tag">Dashboard</span>
      <h3>儀表板版</h3>
      <p>互動面板，快速瀏覽功效與臨床數據</p>
    </a>
    <a href="b2c-c.html" class="card">
      <span class="style-tag">Showcase</span>
      <h3>產品展示版</h3>
      <p>視覺衝擊力強，消化道旅程動畫</p>
    </a>
    <div class="section-label">B2B 專業版</div>
    <a href="b2b-a.html" class="card">
      <span class="style-tag">Scrollytelling</span>
      <h3>滾動敘事版</h3>
      <p>從市場洞察到科學驗證的完整故事線</p>
    </a>
    <a href="b2b-b.html" class="card">
      <span class="style-tag">Dashboard</span>
      <h3>儀表板版</h3>
      <p>左側導航，快速切換菌株、實驗、臨床、配方數據</p>
    </a>
    <a href="b2b-c.html" class="card">
      <span class="style-tag">Showcase</span>
      <h3>產品展示版</h3>
      <p>高端品牌感，圖表輪播與數據亮點</p>
    </a>
  </div>
</body>
</html>
```

- [ ] **Step 2: Verify all pages**

Open each page and check:
```bash
open index.html
```
Click each link, verify:
1. All 6 pages load without JS errors (check browser console)
2. d3.js charts render in all pages
3. Interactive elements work (hover, click, dropdown, toggle)
4. Responsive layout on resize
5. Index page links all work

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add index navigation page for all 6 variants"
```
