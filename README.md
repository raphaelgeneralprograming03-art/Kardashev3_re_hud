
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kardashev Type III - Bio-Tactical Dashboard</title>
  <style>
    * {
      box-sizing: border-box;
      user-select: none;
    }
    body, html {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: #030507;
      font-family: 'Consolas', 'Courier New', monospace;
      color: #00ff66;
    }

    #webgl-container {
      width: 100%;
      height: 100%;
      position: absolute;
      top: 0;
      left: 0;
      z-index: 1;
    }

    /* Efeito CRT / Scanlines (Estilo Resident Evil UI) */
    .scanlines {
      position: absolute;
      top: 0; left: 0; width: 100%; height: 100%;
      background: linear-gradient(
        rgba(18, 16, 16, 0) 50%, 
        rgba(0, 0, 0, 0.35) 50%
      );
      background-size: 100% 4px;
      z-index: 10;
      pointer-events: none;
    }

    /* Interface HUD Estilo Resident Evil */
    .re-hud-frame {
      position: absolute;
      top: 0; left: 0; width: 100%; height: 100%;
      z-index: 20;
      pointer-events: none;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      padding: 20px;
    }

    .header-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: rgba(8, 14, 10, 0.85);
      border: 1px solid #00ff66;
      border-left: 8px solid #ff0033;
      padding: 10px 20px;
      box-shadow: 0 0 15px rgba(0, 255, 102, 0.2);
    }

    .status-badge {
      color: #ff0033;
      font-weight: bold;
      animation: blink 1.2s infinite;
    }

    @keyframes blink {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.3; }
    }

    .main-metrics {
      display: flex;
      justify-content: space-between;
      align-items: flex-end;
    }

    .panel-box {
      background: rgba(5, 10, 15, 0.88);
      border: 1px solid #00ff66;
      padding: 15px;
      width: 320px;
      box-shadow: inset 0 0 10px rgba(0, 255, 102, 0.1);
    }

    .panel-title {
      font-size: 13px;
      color: #ffffff;
      background: #ff0033;
      padding: 2px 6px;
      margin-bottom: 10px;
      display: inline-block;
      font-weight: bold;
    }

    .energy-row {
      font-size: 11px;
      margin: 6px 0;
      display: flex;
      justify-content: space-between;
      border-bottom: 1px dashed rgba(0, 255, 102, 0.3);
      padding-bottom: 2px;
    }

    .energy-val {
      color: #ffffff;
      font-weight: bold;
    }

    .crosshair {
      position: absolute;
      top: 50%; left: 50%;
      transform: translate(-50%, -50%);
      width: 40px; height: 40px;
      border: 1px solid rgba(0, 255, 102, 0.4);
      border-radius: 50%;
      pointer-events: none;
      z-index: 15;
    }

    .crosshair::before, .crosshair::after {
      content: '';
      position: absolute;
      background: #ff0033;
    }
    .crosshair::before { top: 19px; left: -10px; width: 60px; height: 2px; }
    .crosshair::after { left: 19px; top: -10px; height: 60px; width: 2px; }
  </style>

  <!-- Importar Three.js e OrbitControls via CDN -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body>

  <div class="scanlines"></div>
  <div class="crosshair"></div>

  <!-- Camada WebGL 3D -->
  <div id="webgl-container"></div>

  <!-- Moldura HUD Resident Evil -->
  <div class="re-hud-frame">
    <div class="header-bar">
      <div>
        <span style="color: #fff; font-size: 16px; font-weight: bold;">SYSTEM DIAGNOSTIC: KARDASHEV TYPE III</span>
        <div style="font-size: 10px; color: #00ff66;">GALACTIC ENERGY HARVESTING NETWORK</div>
      </div>
      <div class="status-badge">[ LEVEL 3 CRITICAL OUTPUT ]</div>
    </div>

    <div class="main-metrics">
      <!-- Painel Esquerdo: Diagnóstico de Energia Planetária e Estelar -->
      <div class="panel-box">
        <div class="panel-title">SUBSYSTEM METRICS</div>
        <div class="energy-row"><span>🌍 TERRA (GEOTHERMAL):</span><span class="energy-val">100% OPTIMAL</span></div>
        <div class="energy-row"><span>🌊 OCEANIC KINETIC:</span><span class="energy-val">4.2 × 10¹⁸ W</span></div>
        <div class="energy-row"><span>☁️ ATMOSPHERIC HARVEST:</span><span class="energy-val">1.8 × 10¹⁹ W</span></div>
        <div class="energy-row"><span>🚀 ORBITAL SOLAR:</span><span class="energy-val">3.8 × 10²⁶ W</span></div>
        <div class="energy-row"><span>⭐ DYSON SWARM NODES:</span><span class="energy-val">100B STARS</span></div>
      </div>

      <!-- Painel Direito: Saída Galáctica Geral -->
      <div class="panel-box">
        <div class="panel-title">GALACTIC SUMMARY</div>
        <div class="energy-row"><span>TOTAL OUTPUT:</span><span class="energy-val" style="color: #ff0033;">10³⁶ WATTS</span></div>
        <div class="energy-row"><span>BLACK HOLE TAP:</span><span class="energy-val">ACTIVE (SAGITTARIUS A*)</span></div>
        <div class="energy-row"><span>QUANTUM LINK:</span><span class="energy-val">STABLE (100%)</span></div>
        <div class="energy-row"><span>BIOMECHANICAL HUD:</span><span class="energy-val">ONLINE</span></div>
      </div>
    </div>
  </div>

  <script>
    // Inicialização do Cenário Three.js 3D
    const container = document.getElementById('webgl-container');
    const scene = new THREE.Scene();
    scene.fog = new THREE.FogExp2(0x030507, 0.0008);

    const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 1, 5000);
    camera.position.set(0, 600, 1200);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    container.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;

    // 1. Criar a Galáxia Espiral 3D (Escala Nível 3 Kardashev)
    const galaxyParticleCount = 80000;
    const galaxyGeometry = new THREE.BufferGeometry();
    const positions = new Float32Array(galaxyParticleCount * 3);
    const colors = new Float32Array(galaxyParticleCount * 3);

    const colorCore = new THREE.Color(0xff0055);
    const colorArm = new THREE.Color(0x00ffaa);

    for (let i = 0; i < galaxyParticleCount; i++) {
      // Distribuição em espiral
      const radius = Math.random() * 800;
      const spinAngle = radius * 0.008;
      const arms = (i % 4) * ((Math.PI * 2) / 4);

      const x = Math.cos(spinAngle + arms) * radius + (Math.random() - 0.5) * 40;
      const y = (Math.random() - 0.5) * (150 - radius * 0.1);
      const z = Math.sin(spinAngle + arms) * radius + (Math.random() - 0.5) * 40;

      positions[i * 3] = x;
      positions[i * 3 + 1] = y;
      positions[i * 3 + 2] = z;

      // Cor com base na distância do centro
      const mixedColor = colorCore.clone().lerp(colorArm, radius / 800);
      colors[i * 3] = mixedColor.r;
      colors[i * 3 + 1] = mixedColor.g;
      colors[i * 3 + 2] = mixedColor.b;
    }

    galaxyGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    galaxyGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

    const galaxyMaterial = new THREE.PointsMaterial({
      size: 3,
      vertexColors: true,
      transparent: true,
      opacity: 0.85,
      blending: THREE.AdditiveBlending
    });

    const galaxyPoints = new THREE.Points(galaxyGeometry, galaxyMaterial);
    scene.add(galaxyPoints);

    // 2. Núcleo Galáctico Supermassivo (Buraco Negro / Coletor Principal)
    const coreGeo = new THREE.SphereGeometry(60, 32, 32);
    const coreMat = new THREE.MeshBasicMaterial({ color: 0xffffff });
    const coreMesh = new THREE.Mesh(coreGeo, coreMat);
    scene.add(coreMesh);

    // Anel de Acreção Radiante
    const ringGeo = new THREE.RingGeometry(70, 140, 64);
    const ringMat = new THREE.MeshBasicMaterial({ color: 0xff0055, side: THREE.DoubleSide, transparent: true, opacity: 0.7 });
    const ringMesh = new THREE.Mesh(ringGeo, ringMat);
    ringMesh.rotation.x = Math.PI / 2;
    scene.add(ringMesh);

    // 3. Planeta Terra Holográfico (Simulação Interna dos Recursos Terra/Mar/Ar)
    const earthGeo = new THREE.SphereGeometry(80, 32, 32);
    const earthMat = new THREE.MeshBasicMaterial({ color: 0x00ffaa, wireframe: true, transparent: true, opacity: 0.4 });
    const earthMesh = new THREE.Mesh(earthGeo, earthMat);
    earthMesh.position.set(-600, 200, 200);
    scene.add(earthMesh);

    // Redes de Energia e Feixes Galácticos
    const lineMaterial = new THREE.LineBasicMaterial({ color: 0x00ff66, transparent: true, opacity: 0.3 });
    const lineGeometry = new THREE.BufferGeometry();
    const linePoints = [
      earthMesh.position,
      coreMesh.position
    ];
    lineGeometry.setFromPoints(linePoints);
    const energyBeam = new THREE.Line(lineGeometry, lineMaterial);
    scene.add(energyBeam);

    // Redimensionamento de Janela
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // Loop de Animação 3D
    function animate() {
      requestAnimationFrame(animate);

      // Rotação da Galáxia
      galaxyPoints.rotation.y += 0.0008;
      ringMesh.rotation.z -= 0.002;
      earthMesh.rotation.y += 0.005;

      controls.update();
      renderer.render(scene, camera);
    }

    animate();
  </script>
</body>
</html>
