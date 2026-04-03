<!DOCTYPE html>
<html lang="en" data-theme="light">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Global Free Trade Zones — Illicit Network Map</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300..600&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/topojson-client@3"></script>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --color-bg: #f7f6f2;
    --color-surface: #ffffff;
    --color-border: #d4d1ca;
    --color-text: #28251d;
    --color-text-muted: #7a7974;
    --color-text-faint: #bab9b4;
    --color-primary: #01696f;
    --color-surface-offset: #f0ede8;
    --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    --radius-md: 0.5rem;
    --radius-lg: 0.75rem;
    --radius-full: 9999px;
    --shadow-md: 0 4px 12px rgba(0,0,0,0.08);
    --shadow-lg: 0 12px 32px rgba(0,0,0,0.12);
    --transition: 180ms cubic-bezier(0.16, 1, 0.3, 1);
  }

  [data-theme="dark"] {
    --color-bg: #171614;
    --color-surface: #1c1b19;
    --color-border: #393836;
    --color-text: #cdccca;
    --color-text-muted: #797876;
    --color-text-faint: #5a5957;
    --color-primary: #4f98a3;
    --color-surface-offset: #22211f;
    --shadow-md: 0 4px 12px rgba(0,0,0,0.3);
    --shadow-lg: 0 12px 32px rgba(0,0,0,0.4);
  }

  body {
    font-family: var(--font-sans);
    background: var(--color-bg);
    color: var(--color-text);
    min-height: 100vh;
    transition: background var(--transition), color var(--transition);
  }

  header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16px 24px;
    border-bottom: 1px solid var(--color-border);
    background: var(--color-surface);
  }

  .header-left h1 {
    font-size: 15px;
    font-weight: 600;
    letter-spacing: -0.01em;
    color: var(--color-text);
  }

  .header-left p {
    font-size: 12px;
    color: var(--color-text-muted);
    margin-top: 2px;
  }

  .theme-toggle {
    background: none;
    border: 1px solid var(--color-border);
    border-radius: var(--radius-md);
    padding: 6px 8px;
    cursor: pointer;
    color: var(--color-text-muted);
    display: flex;
    align-items: center;
    transition: all var(--transition);
  }

  .theme-toggle:hover {
    background: var(--color-surface-offset);
    color: var(--color-text);
  }

  .map-container {
    padding: 20px 24px;
    max-width: 1100px;
    margin: 0 auto;
  }

  .map-wrapper {
    position: relative;
    background: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-lg);
    overflow: hidden;
    box-shadow: var(--shadow-md);
  }

  #map { width: 100%; display: block; }

  .tooltip-box {
    position: absolute;
    background: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: var(--radius-md);
    padding: 10px 14px;
    font-size: 12px;
    color: var(--color-text);
    pointer-events: none;
    opacity: 0;
    transition: opacity 0.15s;
    max-width: 280px;
    line-height: 1.55;
    z-index: 10;
    box-shadow: var(--shadow-lg);
  }

  .controls-row {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 12px 20px;
    padding: 14px 16px;
    border-top: 1px solid var(--color-border);
    background: var(--color-surface-offset);
  }

  .legend {
    display: flex;
    flex-wrap: wrap;
    gap: 8px 16px;
    flex: 1;
  }

  .legend-item {
    display: flex;
    align-items: center;
    gap: 5px;
    font-size: 11.5px;
    color: var(--color-text-muted);
  }

  .legend-dot {
    width: 9px;
    height: 9px;
    border-radius: 50%;
    flex-shrink: 0;
  }

  .filter-group {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
  }

  .filter-group button {
    font-size: 11.5px;
    font-family: var(--font-sans);
    padding: 4px 11px;
    border-radius: var(--radius-full);
    border: 1px solid var(--color-border);
    background: var(--color-surface);
    color: var(--color-text-muted);
    cursor: pointer;
    transition: all var(--transition);
  }

  .filter-group button:hover {
    background: var(--color-surface-offset);
    color: var(--color-text);
  }

  .filter-group button.active {
    background: var(--color-primary);
    color: #fff;
    border-color: var(--color-primary);
  }

  footer {
    text-align: center;
    padding: 16px;
    font-size: 11px;
    color: var(--color-text-faint);
  }
</style>
</head>
<body>
<header>
  <div class="header-left">
    <h1>Global Free Trade Zones — Illicit Network Map</h1>
    <p>Hover over a marker for details. Click a region filter to focus.</p>
  </div>
  <button class="theme-toggle" data-theme-toggle aria-label="Toggle dark mode">
    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/></svg>
  </button>
</header>

<div class="map-container">
  <div class="map-wrapper">
    <div style="position:relative">
      <svg id="map" viewBox="0 0 760 520"></svg>
      <div class="tooltip-box" id="tip"></div>
    </div>

    <div class="controls-row">
      <div class="legend">
        <div class="legend-item"><div class="legend-dot" style="background:#D85A30"></div>Latin America</div>
        <div class="legend-item"><div class="legend-dot" style="background:#1D9E75"></div>Caribbean / Central America</div>
        <div class="legend-item"><div class="legend-dot" style="background:#534AB7"></div>West Africa</div>
        <div class="legend-item"><div class="legend-dot" style="background:#378ADD"></div>North America / Europe</div>
        <div class="legend-item"><div class="legend-dot" style="background:#D4537E"></div>Middle East</div>
      </div>
      <div class="filter-group" id="controls">
        <button class="active" onclick="filterZones('all',this)">All zones</button>
        <button onclick="filterZones('latam',this)">Latin America</button>
        <button onclick="filterZones('carib',this)">Caribbean</button>
        <button onclick="filterZones('wafrica',this)">West Africa</button>
        <button onclick="filterZones('north',this)">North America / EU</button>
        <button onclick="filterZones('mideast',this)">Middle East</button>
      </div>
    </div>
  </div>
</div>

<footer>Data sourced from U.S. State Dept (INCSR), UNODC, FATF, FinCEN, U.S. Treasury (OFAC), DOJ, DEA, INTERPOL, TRACIT, and congressional records.</footer>

<script>
(function(){
  const t = document.querySelector('[data-theme-toggle]');
  const r = document.documentElement;
  let d = matchMedia('(prefers-color-scheme:dark)').matches ? 'dark' : 'light';
  r.setAttribute('data-theme', d);
  t.innerHTML = d === 'dark'
    ? '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="5"/><path d="M12 1v2M12 21v2M4.22 4.22l1.42 1.42M18.36 18.36l1.42 1.42M1 12h2M21 12h2M4.22 19.78l1.42-1.42M18.36 5.64l1.42-1.42"/></svg>'
    : '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/></svg>';
  t.addEventListener('click', () => {
    d = d === 'dark' ? 'light' : 'dark';
    r.setAttribute('data-theme', d);
    t.innerHTML = d === 'dark'
      ? '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="5"/><path d="M12 1v2M12 21v2M4.22 4.22l1.42 1.42M18.36 18.36l1.42 1.42M1 12h2M21 12h2M4.22 19.78l1.42-1.42M18.36 5.64l1.42-1.42"/></svg>'
      : '<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/></svg>';
  });
})();

const zones = [
  {name:"Colon Free Zone",lat:9.35,lon:-79.9,region:"carib",color:"#1D9E75",r:5.5,
    desc:"World's 2nd-largest FTZ (after Hong Kong). 2,500+ companies, $19B annual turnover. FinCEN flagged shipments through here as BMPE red flags. TRACIT profiled as key illicit trade corridor. Used by Colombian & Mexican cartels for trade-based money laundering.",
    lx:10,ly:-10,anchor:"start"},

  {name:"Margarita Island FTZ",lat:11.0,lon:-63.9,region:"carib",color:"#1D9E75",r:4.5,
    desc:"Venezuelan duty-free island. U.S. Congressional testimony: Hezbollah operates training camps & smuggling networks here. Former Treasury official called it 'a center of gravity' for Hezbollah in the Western Hemisphere. DOJ alleges Venezuelan officials recruited Hezbollah/Hamas members for clandestine camps. 10,400+ Venezuelan passports issued to Lebanese/Syrian/Iranian nationals (2010–2019).",
    lx:8,ly:-8,anchor:"start"},

  {name:"Curacao FTZ",lat:12.17,lon:-68.98,region:"carib",color:"#1D9E75",r:4,
    desc:"Dutch Caribbean free zone. In April 2009, 250 law enforcement officials arrested 17 suspects in a Hezbollah-linked drug ring shipping 2,000+ kg cocaine/year to Europe & Middle East. Proceeds funneled to Hezbollah via informal banking (Jamestown Foundation, RAND).",
    lx:-8,ly:-10,anchor:"end"},

  {name:"Trinidad & Tobago",lat:10.5,lon:-61.3,region:"carib",color:"#1D9E75",r:3.5,
    desc:"Point Lisas free zone. 7 miles from Venezuela. State of emergency declared Jan 2025 over record narco-violence (61 homicides in Dec 2024 alone). Cocaine enters via small boats from Venezuela. 132 known coastal disembarkation points (Min. of National Security). U.S. State Dept: key transshipment location for cocaine to US/UK/Europe.",
    lx:8,ly:12,anchor:"start"},

  {name:"Kingston FTZ",lat:17.97,lon:-76.79,region:"carib",color:"#1D9E75",r:4,
    desc:"Jamaica's main FTZ. 150+ unofficial coastal entry points. 1,500kg cocaine seized at Kingston Freeport Terminal (Jan 2023, $80M — Jamaica Constabulary Force). 538kg disguised as Cuban honey seized March 2025. Key corridor on Cartagena-to-US/UK cocaine route. Jamaica PM: 'increasingly important drug transshipment location.'",
    lx:-8,ly:-8,anchor:"end"},

  {name:"Manaus Free Zone",lat:-3.12,lon:-60.02,region:"latam",color:"#D85A30",r:4.5,
    desc:"Brazil's largest FTZ, at the confluence of the Solimões & Negro rivers. UNODC: cocaine from Colombia/Peru arrives via river, is loaded onto cargo ships at Manaus heading to Africa & Europe. INTERPOL established Amazon-focused task force based here (Feb 2026). Brazilian govt (2025): criminal gangs (Red Command, PCC) now operate in 45% of Amazon municipalities. Over 900 drug routes transit the Amazon Basin.",
    lx:8,ly:-8,anchor:"start"},

  {name:"Tri-Border Area",lat:-25.5,lon:-54.6,region:"latam",color:"#D85A30",r:5.5,
    desc:"Argentina-Brazil-Paraguay junction. FinCEN: Hezbollah generates revenue here through money laundering, narcotics trafficking, charcoal/oil smuggling, counterfeiting & illicit diamond trade. U.S. State Dept Rewards for Justice: $10M reward for info on Hezbollah financial networks in the TBA (May 2025). Active since the 1980s. Assad Barakat network sanctioned by U.S. Treasury.",
    lx:8,ly:4,anchor:"start"},

  {name:"Iquique FTZ",lat:-20.21,lon:-70.15,region:"latam",color:"#D85A30",r:4,
    desc:"Chile's largest FTZ (ZOFRI). 1,700+ businesses, 30,000 jobs, trading with 70+ countries. U.S. State Dept INCSR flagged Chile's FTZs as 'largely unregulated' for AML. Major counterfeiting pipeline for Chinese goods to Bolivia/Paraguay. FBI-prompted takedown of 'Clan Cheng' ($200M fraud) in 2026. Chilean prosecutors investigating Lebanese-linked firms for Hezbollah money laundering (2024).",
    lx:-8,ly:4,anchor:"end"},

  {name:"Buenaventura",lat:3.88,lon:-77.02,region:"latam",color:"#D85A30",r:4.5,
    desc:"Colombia's Pacific port. DOJ: Clan del Golfo's primary income is cocaine trafficking — controlled deliveries of 191kg & 172kg made in Cartagena/Valledupar (2018). Key transit for cocaine to Central America & Mexico. Crisis Group identifies Buenaventura as one of Latin America's worst-hit drug trafficking hotspots with extreme urban violence.",
    lx:-8,ly:4,anchor:"end"},

  {name:"Cartagena FTZ",lat:10.39,lon:-75.51,region:"latam",color:"#D85A30",r:4,
    desc:"Colombia's Caribbean port zone. ICE: traffickers used parasitic devices on cargo ship hulls at Cartagena, with insider access to port scheduling data. Crisis Group: established Cartagena-to-Miami cocaine corridor. FinCEN: Black Market Peso Exchange & trade-based money laundering node. DEA agent (Irizarry) corruption case exposed deep penetration of port logistics.",
    lx:-8,ly:4,anchor:"end"},

  {name:"Lome Free Zone",lat:6.13,lon:1.22,region:"wafrica",color:"#534AB7",r:4.5,
    desc:"Togo's port zone. On the Lagos–Cotonou–Lomé–Accra trafficking corridor (~500km, UNODC). FATF (2016): Gulf of Guinea & Sahel trafficking routes are 'so heavily used' they finance terrorist groups including Boko Haram & AQIM. U.S. Treasury identified Togo-Ghana border in a Hezbollah-linked drug/cash laundering route — products moved across borders to Accra airport, cash shipped via MEA to Lebanon. Lomé airport flagged by UNODC for cocaine couriers to Europe.",
    lx:-8,ly:-10,anchor:"end"},

  {name:"Lagos Free Zone",lat:6.45,lon:3.39,region:"wafrica",color:"#534AB7",r:5,
    desc:"Nigeria's commercial hub. UNODC: Lagos airport is a major transit point for cocaine couriers. GI-TOC (2025): up to 30% of Europe's cocaine transits West Africa. FATF (2016): Boko Haram finances through extortion, ransom, cattle rustling & trade in goods; ISWAP (ISIS affiliate, 3,500–5,000 fighters — UN Security Council) exploits cash economy via kidnapping & illegal levies. Nigerian FIU detected ISIS-linked crypto transfers to two Nigerians ($19,900 & $9,900). U.S. Treasury: Hezbollah used-car/cash laundering through West African networks.",
    lx:8,ly:12,anchor:"start"},

  {name:"Dakar FTZ",lat:14.69,lon:-17.44,region:"wafrica",color:"#534AB7",r:4.5,
    desc:"Senegal's port zone. GI-TOC (Sept 2025): Port of Dakar is West Africa's 'most strategic hub' for cocaine exports to Spain/Europe. Balkan organized crime groups (Kavač, Škaljari clans) increasingly active. On 'Highway 10' — the shortest transatlantic route from Brazil (~2,500km). FATF (2016): Sahel & Gulf of Guinea trafficking routes used to finance Boko Haram & AQIM. GIABA (West African AML body) headquartered in Dakar. ~30,000 Lebanese residents. HSI provided TBML training to Senegalese law enforcement (INCSR 2025).",
    lx:8,ly:-10,anchor:"start"},

  {name:"Abidjan FTZ",lat:5.36,lon:-4.01,region:"wafrica",color:"#534AB7",r:4.5,
    desc:"Ivory Coast. Est. 60,000–300,000 Lebanese residents (largest Lebanese diaspora in West Africa). FDD: Hezbollah has infiltrated diaspora institutions from Abidjan to Foz do Iguaçu since the 1980s. 1.2 tonnes of cocaine destined for Abidjan seized at Santos port, Brazil (2018). UNODC: Abidjan is now a destination market, not just transit, with growing local user communities.",
    lx:-8,ly:12,anchor:"end"},

  {name:"Accra FTZ",lat:5.56,lon:-0.19,region:"wafrica",color:"#534AB7",r:3.5,
    desc:"Ghana's free zone. U.S. Treasury: Accra airport identified as endpoint in Hezbollah-linked used-car/cash laundering scheme (drugs transported across Togo-Ghana border, cash flown to Lebanon). Ghana govt survey (2021): cocaine is most-abused drug in Greater Accra. UNODC: Ghana is a significant synthetic drug producer & cocaine transit zone to UK. Host of 2025 'Accra Call to Action' drug summit.",
    lx:8,ly:-4,anchor:"start"},

  {name:"Port of Montreal",lat:45.5,lon:-73.56,region:"north",color:"#378ADD",r:5,
    desc:"Canada. Canadian govt 2025 risk report: Port of Montreal is a 'known link where luxury vehicles are shipped to Lebanon, financially supporting Hezbollah.' Operation Vector (2024): 598 stolen vehicles worth $34.5M seized (OPP/CBSA). In 2023, 1,200 stolen vehicles recovered at the port. DEA (2008): identified Hezbollah operatives in Montreal via wiretap evidence from Colombian narco-kingpins.",
    lx:8,ly:-10,anchor:"start"},

  {name:"Halifax Port",lat:44.65,lon:-63.57,region:"north",color:"#378ADD",r:4,
    desc:"Canada. CBSA (March 2024): 1,556kg cocaine seized, valued at $194M, from a container en route from California to Europe via Panama Canal. DEA identified Halifax as part of Hezbollah 'leadership cells for Western Hemisphere operations' (2008 briefing to RCMP). Canadian 2025 risk assessment confirmed Hezbollah's use of both licit and illicit channels in Canada.",
    lx:8,ly:6,anchor:"start"},

  {name:"Miami FTZ",lat:25.76,lon:-80.19,region:"north",color:"#378ADD",r:4.5,
    desc:"US gateway to Latin America & Caribbean. FinCEN: primary hub for the Black Market Peso Exchange — 'one of the most extensive money laundering methodologies' in the Western Hemisphere. FinCEN Advisory (2010): flagged shipments of electronics, auto parts & precious metals from Miami to FTZs as TBML red flags. DOJ: $50M+ in drug proceeds laundered through BMPE via Miami bank accounts (Operation Mallorca).",
    lx:8,ly:-10,anchor:"start"},

  {name:"Rotterdam Port",lat:51.92,lon:4.48,region:"north",color:"#378ADD",r:4,
    desc:"Europe's largest port. EU member states reported a record 356 tonnes of cocaine seized in 2022, with Belgium, Spain & the Netherlands reporting highest volumes. Europol: key entry point for Latin American cocaine. Crisis Group: established trafficking corridor from Buenaventura & Cartagena. Multiple 'super cartel' arrests linked to Rotterdam trafficking operations.",
    lx:8,ly:-8,anchor:"start"},

  {name:"Port of Beirut",lat:33.9,lon:35.5,region:"mideast",color:"#D4537E",r:5,
    desc:"Lebanon. Hezbollah's home base. FinCEN (2024): Hezbollah generates ~$1B annually from Iranian support, international businesses, donor networks, corruption & money laundering. U.S. Treasury: destination for stolen vehicles from Canada, laundered funds & trade flows from Latin America, West Africa & the Caribbean. Multiple OFAC designations target Hezbollah financiers based in Lebanon.",
    lx:8,ly:-10,anchor:"start"},

  {name:"Dubai / Jebel Ali",lat:25.0,lon:55.1,region:"mideast",color:"#D4537E",r:4.5,
    desc:"UAE free zone. FATF (2020): JAFZA accounts for 32% of UAE foreign investment & 24% of Dubai's GDP ($118B trade value) — yet only 99 financial intelligence requests from free zones over 5 years. U.S. State Dept INCSR: UAE listed as 'major money laundering country.' OFAC designated multiple Jebel Ali entities for Iran/Russia sanctions evasion (Bitubiz, Hennesea). U.S. Treasury (2022): OFAC sanctioned six individuals after UAE court convicted them for transferring $782,000 from Dubai to Boko Haram in Nigeria. Europol: European 'super cartel' leaders operated here. UAE removed from FATF grey list Feb 2024.",
    lx:8,ly:6,anchor:"start"},
];

const routes = [
  {from:"Colon Free Zone",to:"Port of Beirut"},
  {from:"Tri-Border Area",to:"Port of Beirut"},
  {from:"Margarita Island FTZ",to:"Port of Beirut"},
  {from:"Port of Montreal",to:"Port of Beirut"},
  {from:"Lagos Free Zone",to:"Port of Beirut"},
  {from:"Abidjan FTZ",to:"Port of Beirut"},
  {from:"Colon Free Zone",to:"Miami FTZ"},
  {from:"Buenaventura",to:"Colon Free Zone"},
  {from:"Buenaventura",to:"Rotterdam Port"},
  {from:"Port of Beirut",to:"Dubai / Jebel Ali"},
  {from:"Dakar FTZ",to:"Rotterdam Port"},
  {from:"Lagos Free Zone",to:"Dakar FTZ"},
  {from:"Port of Montreal",to:"Rotterdam Port"},
  {from:"Halifax Port",to:"Port of Montreal"},
];

function getColors() {
  const dark = document.documentElement.getAttribute('data-theme') === 'dark';
  return {
    landFill: dark ? '#2a2a28' : '#e8e6df',
    seaFill: dark ? '#1a1a19' : '#f4f3ef',
    borderStroke: dark ? '#3a3a37' : '#cccbc4',
    routeColor: dark ? 'rgba(255,255,255,0.1)' : 'rgba(0,0,0,0.07)',
    labelColor: dark ? '#b0afa8' : '#5f5e5a',
  };
}

const svg = d3.select('#map');
const w = 760, h = 520;
const proj = d3.geoNaturalEarth1().center([-10, 12]).scale(195).translate([w / 2, h / 2]);
const path = d3.geoPath().projection(proj);

let colors = getColors();
const seaRect = svg.append('rect').attr('width', w).attr('height', h).attr('fill', colors.seaFill).attr('rx', 8);
let landG, routeG, dotG, labelG;

function buildMap(world) {
  colors = getColors();
  seaRect.attr('fill', colors.seaFill);
  if (landG) landG.remove();
  if (routeG) routeG.remove();
  if (dotG) dotG.remove();
  if (labelG) labelG.remove();

  const land = topojson.feature(world, world.objects.countries);
  landG = svg.append('g');
  landG.selectAll('path').data(land.features).join('path')
    .attr('d', path).attr('fill', colors.landFill).attr('stroke', colors.borderStroke).attr('stroke-width', 0.3);

  routeG = svg.append('g');
  routes.forEach(r => {
    const from = zones.find(z => z.name === r.from);
    const to = zones.find(z => z.name === r.to);
    if (!from || !to) return;
    const [x1, y1] = proj([from.lon, from.lat]);
    const [x2, y2] = proj([to.lon, to.lat]);
    const mx = (x1 + x2) / 2, my = (y1 + y2) / 2 - Math.abs(x2 - x1) * 0.15;
    routeG.append('path')
      .attr('d', `M${x1},${y1} Q${mx},${my} ${x2},${y2}`)
      .attr('fill', 'none').attr('stroke', colors.routeColor).attr('stroke-width', 1)
      .attr('stroke-dasharray', '5,4')
      .attr('class', 'route-line')
      .attr('data-from', r.from).attr('data-to', r.to);
  });

  const tip = document.getElementById('tip');
  dotG = svg.append('g');
  labelG = svg.append('g');

  zones.forEach(z => {
    const [cx, cy] = proj([z.lon, z.lat]);
    if (cx < 0 || cx > w || cy < 0 || cy > h) return;
    z.cx = cx; z.cy = cy;

    const g = dotG.append('g').attr('data-region', z.region).style('cursor', 'pointer');
    g.append('circle').attr('cx', cx).attr('cy', cy).attr('r', z.r + 4).attr('fill', z.color).attr('opacity', 0.15);
    g.append('circle').attr('cx', cx).attr('cy', cy).attr('r', z.r).attr('fill', z.color).attr('opacity', 0.9).attr('stroke', '#fff').attr('stroke-width', 0.8);

    const lx = cx + z.lx, ly = cy + z.ly;
    const label = labelG.append('g').attr('data-region', z.region).attr('data-name', z.name);
    label.append('line').attr('x1', cx).attr('y1', cy).attr('x2', lx).attr('y2', ly)
      .attr('stroke', colors.labelColor).attr('stroke-width', 0.4).attr('opacity', 0.5);
    label.append('text')
      .attr('x', z.anchor === 'end' ? lx - 3 : lx + 3)
      .attr('y', ly)
      .attr('text-anchor', z.anchor)
      .attr('dominant-baseline', 'central')
      .attr('font-size', '9.5px')
      .attr('font-family', "'Inter', sans-serif")
      .attr('fill', colors.labelColor)
      .attr('font-weight', 400)
      .text(z.name);

    g.on('mouseenter', function(e) {
      d3.select(this).selectAll('circle').filter((d, i) => i === 1).attr('r', z.r + 2);
      tip.innerHTML = `<div style="font-weight:600;margin-bottom:4px;color:${z.color}">${z.name}</div>${z.desc}`;
      tip.style.opacity = 1;
      const pb = document.querySelector('[style*="position:relative"]').getBoundingClientRect();
      const bx = e.clientX - pb.left, by = e.clientY - pb.top;
      let left = bx + 14, top = by - 10;
      if (left + 280 > w) left = bx - 294;
      if (top + 100 > h) top = top - 80;
      if (top < 0) top = 10;
      tip.style.left = left + 'px'; tip.style.top = top + 'px';
      svg.selectAll('.route-line').each(function() {
        const el = d3.select(this);
        if (el.attr('data-from') === z.name || el.attr('data-to') === z.name)
          el.attr('stroke', z.color).attr('stroke-width', 1.8).attr('stroke-opacity', 0.55);
      });
      labelG.select(`[data-name="${z.name}"] text`).attr('font-weight', 600).attr('font-size', '11px');
    })
    .on('mouseleave', function() {
      d3.select(this).selectAll('circle').filter((d, i) => i === 1).attr('r', z.r);
      tip.style.opacity = 0;
      svg.selectAll('.route-line').attr('stroke', colors.routeColor).attr('stroke-width', 1).attr('stroke-opacity', 1);
      labelG.select(`[data-name="${z.name}"] text`).attr('font-weight', 400).attr('font-size', '9.5px');
    });
  });
}

let worldData;
fetch('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json')
  .then(r => r.json())
  .then(world => { worldData = world; buildMap(world); });

document.querySelector('[data-theme-toggle]').addEventListener('click', () => {
  if (worldData) setTimeout(() => buildMap(worldData), 50);
});

window.filterZones = function(region, btn) {
  document.querySelectorAll('#controls button').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  svg.selectAll('g[data-region]').each(function() {
    const el = d3.select(this);
    const show = region === 'all' || el.attr('data-region') === region;
    el.transition().duration(300).attr('opacity', show ? 1 : 0.08);
  });
  svg.selectAll('.route-line').transition().duration(300).attr('opacity', region === 'all' ? 1 : 0.15);
};
</script>
</body>
</html>
