<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>미래실험실: 혜화동 도시 생태계 원형 포함 지도</title>
  <script src="https://d3js.org/d3.v7.min.js"></script>
  <style>
    body { margin: 0; display: flex; justify-content: center; background-color: #f8f9fa; font-family: 'Pretendard', sans-serif; }
    svg { max-width: 100%; height: auto; cursor: pointer; }
    .node { cursor: pointer; }
    .node:hover { stroke: #000; stroke-width: 1.5px; }
    .label { font-size: 10px; text-anchor: middle; fill: #333; font-weight: 600; pointer-events: none; text-shadow: 0 1px 3px rgba(255,255,255,0.8); }
  </style>
</head>
<body>

<div id="chart"></div>

<script>
  // 1. 혜화동 레이어 데이터 구조화 (JSON 트리)
  // 사용자가 제공한 <Final map structure> 데이터를 기반으로 구축했습니다.
  const data = {
    name: "혜화동",
    children: [
      {
        name: "LAYER 1B — INFRASTRUCTURE",
        children: [
          {
            name: "Medical",
            children: [
              { name: "SNU Hospital (Main)", value: 1 },
              { name: "SNU Dental Hospital", value: 1 },
              { name: "SNU Hospital Annex", value: 1 },
              { name: "SNU Cancer Hospital", value: 1 },
              { name: "Local Clinic - Internal Medicine", value: 1 },
              { name: "Local Clinic - Dermatology", value: 1 },
              { name: "Local Clinic - Orthopedics", value: 1 }
            ]
          },
          {
            name: "Education",
            children: [
              { name: "Sungkyunkwan University", value: 1 },
              { name: "Hongik Univ Daehak-ro Campus", value: 1 },
              { name: "KNOU Headquarters", value: 1 },
              { name: "SNU Attached Elementary", value: 1 },
              { name: "SNU Attached Middle School", value: 1 },
              { name: "SNU Attached Girls Middle", value: 1 },
              { name: "Seoul Hyoje Elementary", value: 1 },
              { name: "Catholic Univ (nearby)", value: 1 }
            ]
          },
          {
            name: "Life Service",
            children: [
              {
                name: "Public Institution (Admin)",
                children: [
                  { name: "Jongno-gu Office", value: 1 },
                  { name: "Hyehwa-dong Community Center", value: 1 },
                  { name: "Jongno Police Station", value: 1 },
                  { name: "Hyehwa Fire Station", value: 1 },
                  { name: "Jongno Post Office (Hyehwa)", value: 1 },
                  { name: "Hyehwa Post Office", value: 1 },
                  { name: "National Election Commission", value: 1 },
                  { name: "Military Manpower Admin", value: 1 },
                  { name: "Naksan Park", value: 1 },
                  { name: "Marronnier Park", value: 1 },
                  { name: "Marronnier Park Outdoor Stage", value: 1 }
                ]
              },
              {
                name: "Daily Convenience Facility",
                children: [
                  { name: "KB Bank ATM", value: 1 },
                  { name: "Shinhan Bank", value: 1 },
                  { name: "iPhone Fix", value: 1 },
                  { name: "Youth Phone", value: 1 },
                  { name: "Optician cluster", value: 1 },
                  { name: "Hair salon cluster", value: 1 },
                  { name: "Dermatology cluster", value: 1 },
                  { name: "Dental clinic", value: 1 },
                  { name: "Print shop", value: 1 },
                  { name: "Coin laundry", value: 1 },
                  { name: "Cafe24 Business Center", value: 1 },
                  { name: "Tutoring academies", value: 1 },
                  { name: "Real estate cluster", value: 1 },
                  { name: "Goshiwon cluster", value: 1 }
                ]
              }
            ]
          },
          {
            name: "Transport",
            children: [
              { name: "Roads", children: [{name: "Daehak-ro", value: 1}, {name: "Dongsung-gil", value: 1}, {name: "Changgyeonggung-ro", value: 1}] },
              { name: "Parking", children: [{name: "Marronnier Parking", value: 1}, {name: "SNU Hospital Parking", value: 1}, {name: "Public Parking Lot", value: 1}] },
              {
                name: "Traffic",
                children: [
                  { name: "Subway", children: [{name: "Hyehwa Station (Line 4)", value: 1}] },
                  { name: "Bus", children: [
                      { name: "Blue Bus (100, 101, ...)", value: 1 },
                      { name: "Green Bus (2112, ...)", value: 1 },
                      { name: "Local Village Bus (Jongno 07, ...)", value: 1 },
                      { name: "Bus station: Exit 4", value: 1 },
                      { name: "Bus station: Exit 1", value: 1 }
                    ] 
                  }
                ]
              }
            ]
          },
          {
            name: "Utility Infrastructure",
            children: [
              { name: "Energy", children: [{name: "KEPCO substation", value: 1}, {name: "Gas regulator", value: 1}] },
              { name: "Water", children: [{name: "Water main", value: 1}, {name: "Sewage line", value: 1}, {name: "Treatment Plant (external)", value: 1}] },
              { name: "Waste", children: [{name: "Waste collection route", value: 1}, {name: "Mapo Resource Recovery (external)", value: 1}] },
              { name: "Telecommunications", value: 1 }
            ]
          }
        ]
      },
      {
        name: "LAYER 2 — INDUSTRY",
        children: [
          {
            name: "Art-Performance",
            children: [
              { name: "Production companies", children: [{name: "OD Musical Company", value: 1}, {name: "CJ ENM Musical Division", value: 1}, {name: "Shownote", value: 1}] },
              { name: "Platform", children: [{name: "NOL Ticket", value: 1}, {name: "YES 24 ticket", value: 1}, {name: "The Musical Magazine", value: 1}] },
              { 
                name: "Theater",
                children: [
                  { name: "Large Theater", children: [{name: "Arko Arts Theater", value: 1}, {name: "Hongik Univ Art Center", value: 1}, {name: "Dongsung Art Center", value: 1}] },
                  { name: "Small theater", children: [{name: "TOM Theater", value: 1}, {name: "Link Arts Center", value: 1}, {name: "소극장 혜화당", value: 1}] }
                ]
              },
              { name: "Practice space", children: [{name: "Studio Ilsangjeok", value: 1}, {name: "Yejon Acting Academy", value: 1}] },
              { name: "Physical & Tech Support", children: [{name: "Rental On", value: 1}, {name: "Costume Kookhyang", value: 1}, {name: "Korean Theater Association", value: 1}] },
              { name: "Transportation", children: [{name: "Set warehouse", value: 1}, {name: "Re:Stage Seoul", value: 1}] }
            ]
          },
          {
            name: "Mutualistic Commerce",
            children: [
              { name: "Eatery", children: [{name: "Restaurant & Bar (Jungdon, ...)", value: 1}, {name: "Cafe (Hakrim Dabang, ...)", value: 1}, {name: "Street Food", value: 1}, {name: "Convenient store", value: 1}] },
              { name: "Exhibition", children: [{name: "Gallery (ARKO Art Center, ...)", value: 1}, {name: "Public Art", value: 1}, {name: "Art supply stores", value: 1}] },
              { name: "Shopping", children: [{name: "Merchandise booth", value: 1}, {name: "Brand Store (Innisfree, Daiso)", value: 1}, {name: "Independent Store (Mapi&nd, ...)", value: 1}] },
              { name: "Heritage", children: [{name: "Changgyeonggung Palace", value: 1}, {name: "Seoul Fortress Wall", value: 1}, {name: "Ihwa Mural Village", value: 1}] }
            ]
          }
        ]
      }
    ]
  };

  // 2. 화면 및 D3 설정
  const width = 800;
  const height = width;
  const color = d3.scaleLinear().domain([0, 5]).range(["hsl(210, 30%, 95%)", "hsl(210, 30%, 75%)"]).interpolate(d3.interpolateHcl);

  const pack = data => d3.pack().size([width, height]).padding(3)(d3.hierarchy(data).sum(d => d.value).sort((a, b) => b.value - a.value));
  const root = pack(data);
  let focus = root;
  let view;

  const svg = d3.select("#chart").append("svg")
      .attr("viewBox", `-${width / 2} -${height / 2} ${width} ${height}`)
      .style("display", "block").style("background", "transparent")
      .on("click", (event) => zoom(event, root));

  // 3. 원 그리기
  const node = svg.append("g").selectAll("circle").data(root.descendants().slice(1)).join("circle")
      .attr("fill", d => d.children ? color(d.depth) : "white")
      .attr("pointer-events", d => !d.children ? "none" : null)
      .attr("class", "node")
      .on("mouseover", function() { d3.select(this).attr("stroke", "#000"); })
      .on("mouseout", function() { d3.select(this).attr("stroke", null); })
      .on("click", (event, d) => focus !== d && (zoom(event, d), event.stopPropagation()));

  // 4. 라벨(글씨) 그리기
  const label = svg.append("g").style("font", "10px sans-serif").style("font-weight", "600").attr("pointer-events", "none").attr("text-anchor", "middle")
    .selectAll("text").data(root.descendants()).join("text")
      .style("fill-opacity", d => d.parent === root ? 1 : 0)
      .style("display", d => d.parent === root ? "inline" : "none")
      .text(d => d.data.name);

  // 5. 줌 액션 설정
  zoomTo([root.x, root.y, root.r * 2]);

  function zoomTo(v) {
    const k = width / v[2];
    view = v;
    label.attr("transform", d => `translate(${(d.x - v[0]) * k},${(d.y - v[1]) * k})`);
    node.attr("transform", d => `translate(${(d.x - v[0]) * k},${(d.y - v[1]) * k})`).attr("r", d => d.r * k);
  }

  function zoom(event, d) {
    focus = d;
    const transition = svg.transition().duration(750).tween("zoom", d => {
      const i = d3.interpolateZoom(view, [focus.x, focus.y, focus.r * 2]);
      return t => zoomTo(i(t));
    });
    label.filter(function(d) { return d.parent === focus || this.style.display === "inline"; })
      .transition(transition)
      .style("fill-opacity", d => d.parent === focus ? 1 : 0)
      .on("start", function(d) { if (d.parent === focus) this.style.display = "inline"; })
      .on("end", function(d) { if (d.parent !== focus) this.style.display = "none"; });
  }
</script>
</body>
</html>
