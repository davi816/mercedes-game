<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Mercedes-Benz Racer 3D</title>
  <style>
    body { margin: 0; overflow: hidden; background: black; font-family: sans-serif; }
    canvas { display: block; }

    #menu {
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0, 0, 0, 0.9);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      color: white;
      z-index: 10;
    }

    #logo {
      width: 100px;
      margin-bottom: 20px;
    }

    .car-option {
      background: white;
      color: black;
      padding: 10px 20px;
      margin: 10px;
      border-radius: 10px;
      cursor: pointer;
      font-size: 18px;
    }

    .car-option:hover {
      background: #ddd;
    }

    #score {
      position: absolute;
      top: 10px;
      left: 10px;
      color: white;
      font-size: 18px;
      z-index: 5;
    }

    .mobile-controls {
      position: absolute;
      bottom: 20px;
      width: 100%;
      display: flex;
      justify-content: space-between;
      padding: 0 50px;
      z-index: 10;
    }

    .control-btn {
      background: rgba(255,255,255,0.7);
      border: none;
      border-radius: 50%;
      width: 60px;
      height: 60px;
      font-size: 30px;
      font-weight: bold;
      cursor: pointer;
    }

    @media (min-width: 768px) {
      .mobile-controls { display: none; }
    }
  </style>
</head>
<body>

<div id="menu">
  <img id="logo" src="https://upload.wikimedia.org/wikipedia/commons/9/90/Mercedes-Logo.svg" alt="Mercedes-Benz Logo" />
  <h1>Mercedes-Benz Racer 3D</h1>
  <p>Elige tu modelo de auto Mercedes-Benz</p>
  <div class="car-option" onclick="startGame('silver')">AMG GT (Plateado)</div>
  <div class="car-option" onclick="startGame('black')">G-Class (Negro)</div>
  <div class="car-option" onclick="startGame('red')">Clase A (Rojo)</div>
</div>

<div id="score">Score: 0 | 🏁 Record: 0</div>

<div class="mobile-controls">
  <button class="control-btn" id="leftBtn">⟵</button>
  <button class="control-btn" id="rightBtn">⟶</button>
</div>

<audio id="music" src="https://cdn.pixabay.com/download/audio/2022/03/01/audio_c6ec941f9f.mp3?filename=powerful-sport-action-rock-126725.mp3" autoplay loop></audio>

<script src="https://cdn.jsdelivr.net/npm/three@0.148.0/build/three.min.js"></script>
<script>
  let scene, camera, renderer, car, road, score = 0, highScore = 0;
  let obstacles = [], carColor = 'silver';
  const keys = { ArrowLeft: false, ArrowRight: false };

  if (localStorage.getItem("highScore")) {
    highScore = parseInt(localStorage.getItem("highScore"));
  }

  const scoreEl = document.getElementById('score');

  function startGame(color) {
    carColor = color;
    document.getElementById('menu').style.display = 'none';
    document.getElementById('music').play();
    init();
    animate();
  }

  function init() {
    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 5, 10);
    camera.lookAt(0, 0, 0);

    renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(5, 10, 7.5);
    scene.add(light);
    scene.add(new THREE.AmbientLight(0xffffff, 0.4));

    const groundGeo = new THREE.PlaneGeometry(20, 1000);
    const groundMat = new THREE.MeshStandardMaterial({ color: 0x222222 });
    const ground = new THREE.Mesh(groundGeo, groundMat);
    ground.rotation.x = -Math.PI / 2;
    scene.add(ground);

    const colorMap = { silver: 0xaaaaaa, black: 0x111111, red: 0xaa0000 };
    const carGeo = new THREE.BoxGeometry(1.5, 0.7, 3);
    const carMat = new THREE.MeshStandardMaterial({ color: colorMap[carColor] || 0xffffff });
    car = new THREE.Mesh(carGeo, carMat);
    car.position.y = 0.35;
    scene.add(car);

    // Eventos
    window.addEventListener('keydown', e => keys[e.key] = true);
    window.addEventListener('keyup', e => keys[e.key] = false);
    window.addEventListener('resize', onResize);

    // Botones móviles
    document.getElementById('leftBtn').addEventListener('touchstart', () => keys.ArrowLeft = true);
    document.getElementById('leftBtn').addEventListener('touchend', () => keys.ArrowLeft = false);
    document.getElementById('rightBtn').addEventListener('touchstart', () => keys.ArrowRight = true);
    document.getElementById('rightBtn').addEventListener('touchend', () => keys.ArrowRight = false);
  }

  function onResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  }

  function addObstacle() {
    const geo = new THREE.BoxGeometry(1, 1, 1);
    const mat = new THREE.MeshStandardMaterial({ color: 0xff3333 });
    const obs = new THREE.Mesh(geo, mat);
    obs.position.set((Math.random() - 0.5) * 10, 0.5, car.position.z - 50);
    scene.add(obs);
    obstacles.push(obs);
  }

  function animate() {
    requestAnimationFrame(animate);
    if (!car) return;

    if (keys.ArrowLeft && car.position.x > -9) car.position.x -= 0.2;
    if (keys.ArrowRight && car.position.x < 9) car.position.x += 0.2;

    car.position.z -= 0.5;

    if (Math.random() < 0.05) addObstacle();

    obstacles.forEach((obs, i) => {
      obs.position.z += 0.6;
      if (obs.position.z > car.position.z + 10) {
        scene.remove(obs);
        obstacles.splice(i, 1);
        score++;
        if (score > highScore) {
          highScore = score;
          localStorage.setItem("highScore", highScore);
        }
      }

      if (obs.position.distanceTo(car.position) < 1.2) {
        alert("💥 ¡Choque! Tu puntuación fue: " + score);
        location.reload();
      }
    });

    scoreEl.innerText = `Score: ${score} | 🏁 Record: ${highScore}`;
    camera.position.z = car.position.z + 10;
    renderer.render(scene, camera);
  }
</script>

</body>
</html>
