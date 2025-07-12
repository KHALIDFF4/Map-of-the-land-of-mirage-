<!-- قائمة التصنيع -->
<div id="crafting">
  <h3>🛠️ التصنيع</h3>
  <button onclick="craft('axe')">🔨 اصنع فأس</button>
  <button onclick="craft('fire')">🔥 اصنع نار</button>
  <div id="inventory">📦 الجرد: ...</div>
</div>#crafting {
  position: absolute;
  top: 10px;
  right: 10px;
  background: rgba(0, 0, 0, 0.5);
  padding: 10px;
  border-radius: 8px;
  color: white;
  font-size: 16px;
  z-index: 2;
}

#crafting button {
  margin: 5px 0;
  width: 100%;
  font-size: 16px;
  padding: 5px;
}// الموارد
let inventory = {
  meat: 0,
  hide: 0,
  axe: false,
};

function updateInventoryUI() {
  document.getElementById("inventory").textContent =
    `📦 الجرد: لحم=${inventory.meat}, جلد=${inventory.hide}, فأس=${inventory.axe ? '✅' : '❌'}`;
}

function attack() {
  const attackRange = 5;
  for (let i = 0; i < dinos.length; i++) {
    const dino = dinos[i];
    const dist = dino.position.distanceTo(player.position);
    if (dist < attackRange) {
      dino.health -= 1;
      dino.material.color.set(0xff0000);
      if (dino.health <= 0) {
        scene.remove(dino);
        dinos.splice(i, 1);
        inventory.meat += 1;
        inventory.hide += 1;
        updateInventoryUI();
      }
      break;
    }
  }
}

function craft(item) {
  if (item === "axe") {
    if (inventory.hide >= 2) {
      inventory.hide -= 2;
      inventory.axe = true;
      alert("✅ صنعت فأساً!");
    } else {
      alert("❌ تحتاج إلى 2 جلد.");
    }
  } else if (item === "fire") {
    if (inventory.axe && inventory.hide >= 1 && inventory.meat >= 1) {
      inventory.hide -= 1;
      inventory.meat -= 1;

      const fire = new THREE.Mesh(
        new THREE.ConeGeometry(0.5, 1, 6),
        new THREE.MeshStandardMaterial({ color: 0xff4500 })
      );
      fire.position.set(player.position.x, 0.5, player.position.z - 1);
      scene.add(fire);

      alert("🔥 نار مشتعلة!");
    } else {
      alert("❌ تحتاج فأس و 1 لحم و 1 جلد.");
    }
  }

  updateInventoryUI();
}

updateInventoryUI();<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>لعبة البقاء</title>
  <link rel="stylesheet" href="style.css"/>
</head>
<body>
  <canvas id="gameCanvas"></canvas>
  <div id="hud">
    <div>❤️ <span id="health">100</span></div>
    <div>🍖 <span id="food">100</span></div>
    <div>🌲 <span id="wood">0</span></div>
  </div>
  <script src="game.js"></script>
</body>
</html>body {
  margin: 0;
  padding: 0;
  overflow: hidden;
  background: #333;
  font-family: sans-serif;
  color: white;
}

canvas {
  display: block;
  background: #4cbb17; /* grassy background */
}

#hud {
  position: absolute;
  top: 10px;
  left: 10px;
  background: rgba(0, 0, 0, 0.4);
  padding: 10px;
  border-radius: 8px;
  font-size: 18px;
}const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let player = {
  x: canvas.width / 2,
  y: canvas.height / 2,
  size: 30,
  color: "#fff",
  speed: 3,
  health: 100,
  food: 100,
  wood: 0
};

let trees = [];
let dinos = [];

for (let i = 0; i < 20; i++) {
  trees.push({
    x: Math.random() * 3000 - 1500,
    y: Math.random() * 3000 - 1500
  });
}

for (let i = 0; i < 5; i++) {
  dinos.push({
    x: Math.random() * 3000 - 1500,
    y: Math.random() * 3000 - 1500,
    size: 40,
    color: "brown"
  });
}

let keys = {};

document.addEventListener("keydown", (e) => {
  keys[e.key] = true;
});

document.addEventListener("keyup", (e) => {
  keys[e.key] = false;
});

function update() {
  if (keys["ArrowUp"] || keys["w"]) player.y -= player.speed;
  if (keys["ArrowDown"] || keys["s"]) player.y += player.speed;
  if (keys["ArrowLeft"] || keys["a"]) player.x -= player.speed;
  if (keys["ArrowRight"] || keys["d"]) player.x += player.speed;

  player.food -= 0.01;
  if (player.food <= 0) {
    player.health -= 0.1;
  }

  document.getElementById("health").textContent = Math.floor(player.health);
  document.getElementById("food").textContent = Math.floor(player.food);
  document.getElementById("wood").textContent = player.wood;

  if (player.health <= 0) {
    alert("لقد مت!");
    location.reload();
  }

  // collision with trees
  trees.forEach((tree, i) => {
    const dx = player.x - tree.x;
    const dy = player.y - tree.y;
    const dist = Math.sqrt(dx * dx + dy * dy);
    if (dist < 40 && keys["e"]) {
      player.wood += 1;
      trees.splice(i, 1); // remove tree
    }
  });

  // collision with dinos
  dinos.forEach((dino) => {
    const dx = player.x - dino.x;
    const dy = player.y - dino.y;
    const dist = Math.sqrt(dx * dx + dy * dy);
    if (dist < 50) {
      player.health -= 0.5;
    }
  });
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // translate camera
  ctx.save();
  ctx.translate(canvas.width / 2 - player.x, canvas.height / 2 - player.y);

  // draw trees
  trees.forEach((tree) => {
    ctx.fillStyle = "green";
    ctx.beginPath();
    ctx.arc(tree.x, tree.y, 20, 0, Math.PI * 2);
    ctx.fill();
  });

  // draw dinos
  dinos.forEach((dino) => {
    ctx.fillStyle = dino.color;
    ctx.beginPath();
    ctx.arc(dino.x, dino.y, dino.size / 2, 0, Math.PI * 2);
    ctx.fill();
  });

  // draw player
  ctx.fillStyle = player.color;
  ctx.beginPath();
  ctx.arc(player.x, player.y, player.size / 2, 0, Math.PI * 2);
  ctx.fill();

  ctx.restore();
}

function gameLoop() {
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

gameLoop();<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <title>لعبة بقاء 3D</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div id="hud">
    ❤️ <span id="health">100</span> | 🍖 <span id="food">100</span>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
  <script src="game.js"></script>
</body>
</html>body {
  margin: 0;
  overflow: hidden;
  background: black;
  font-family: sans-serif;
  color: white;
}

#hud {
  position: absolute;
  top: 10px;
  left: 10px;
  font-size: 20px;
  background: rgba(0, 0, 0, 0.5);
  padding: 6px 12px;
  border-radius: 8px;
}# Map-of-the-land-of-mirage-
science fiction game Surv// إعداد المشهد والكاميرا
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb); // Sky blue

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.y = 2;
camera.position.z = 5;

// إعداد الرندر
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// أرضية
const groundGeometry = new THREE.PlaneGeometry(100, 100);
const groundMaterial = new THREE.MeshStandardMaterial({ color: 0x228b22 });
const ground = new THREE.Mesh(groundGeometry, groundMaterial);
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

// إضاءة
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(5, 10, 5);
scene.add(light);

// اللاعب
const player = new THREE.Mesh(
  new THREE.BoxGeometry(1, 2, 1),
  new THREE.MeshStandardMaterial({ color: 0xffffff })
);
player.position.y = 1;
scene.add(player);

// كاميرا تتبع اللاعب
function updateCamera() {
  camera.position.x = player.position.x;
  camera.position.z = player.position.z + 5;
  camera.lookAt(player.position);
}

// أشجار
for (let i = 0; i < 30; i++) {
  const tree = new THREE.Mesh(
    new THREE.CylinderGeometry(0.2, 0.5, 3),
    new THREE.MeshStandardMaterial({ color: 0x8b4513 })
  );
  const leaves = new THREE.Mesh(
    new THREE.SphereGeometry(1.5, 8, 8),
    new THREE.MeshStandardMaterial({ color: 0x006400 })
  );
  tree.position.set(Math.random() * 80 - 40, 1.5, Math.random() * 80 - 40);
  leaves.position.set(tree.position.x, 4, tree.position.z);
  scene.add(tree);
  scene.add(leaves);
}

// ديناصورات بسيطة
const dinos = [];
for (let i = 0; i < 5; i++) {
  const dino = new THREE.Mesh(
    new THREE.BoxGeometry(2, 1, 4),
    new THREE.MeshStandardMaterial({ color: 0x964B00 })
  );
  dino.position.set(Math.random() * 50 - 25, 0.5, Math.random() * 50 - 25);
  scene.add(dino);
  dinos.push(dino);
}

// حركة اللاعب
let keys = {};
document.addEventListener("keydown", (e) => (keys[e.key.toLowerCase()] = true));
document.addEventListener("keyup", (e) => (keys[e.key.toLowerCase()] = false));

// الحالة
let health = 100;
let food = 100;

function updateStatus() {
  document.getElementById("health").textContent = Math.floor(health);
  document.getElementById("food").textContent = Math.floor(food);
}

function animate() {
  requestAnimationFrame(animate);

  let speed = 0.1;
  if (keys["w"]) player.position.z -= speed;
  if (keys["s"]) player.position.z += speed;
  if (keys["a"]) player.position.x -= speed;
  if (keys["d"]) player.position.x += speed;

  food -= 0.02;
  if (food <= 0) health -= 0.1;
  if (health <= 0) {
    alert("لقد مت! أعد تشغيل الصفحة للعب من جديد.");
    location.reload();
  }

  updateCamera();
  updateStatus();
  renderer.render(scene, camera);
}

animate();const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb); // Sky

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.y = 2;
camera.position.z = 5;

const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// الأرض
const groundGeometry = new THREE.PlaneGeometry(100, 100);
const groundMaterial = new THREE.MeshStandardMaterial({ color: 0x228b22 });
const ground = new THREE.Mesh(groundGeometry, groundMaterial);
ground.rotation.x = -Math.PI / 2;
scene.add(ground);

// إضاءة
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(5, 10, 5);
scene.add(light);

// اللاعب
const player = new THREE.Mesh(
  new THREE.BoxGeometry(1, 2, 1),
  new THREE.MeshStandardMaterial({ color: 0xffffff })
);
player.position.y = 1;
scene.add(player);

function updateCamera() {
  camera.position.x = player.position.x;
  camera.position.z = player.position.z + 5;
  camera.lookAt(player.position);
}

// أشجار
for (let i = 0; i < 30; i++) {
  const trunk = new THREE.Mesh(
    new THREE.CylinderGeometry(0.2, 0.5, 3),
    new THREE.MeshStandardMaterial({ color: 0x8b4513 })
  );
  const leaves = new THREE.Mesh(
    new THREE.SphereGeometry(1.5, 8, 8),
    new THREE.MeshStandardMaterial({ color: 0x006400 })
  );
  const x = Math.random() * 80 - 40;
  const z = Math.random() * 80 - 40;
  trunk.position.set(x, 1.5, z);
  leaves.position.set(x, 4, z);
  scene.add(trunk);
  scene.add(leaves);
}

// ديناصورات
const dinos = [];
for (let i = 0; i < 5; i++) {
  const dino = new THREE.Mesh(
    new THREE.BoxGeometry(2, 1, 4),
    new THREE.MeshStandardMaterial({ color: 0x964B00 })
  );
  dino.position.set(Math.random() * 50 - 25, 0.5, Math.random() * 50 - 25);
  scene.add(dino);
  dinos.push(dino);
}

// تحكم
let keys = {};
document.addEventListener("keydown", (e) => (keys[e.key.toLowerCase()] = true));
document.addEventListener("keyup", (e) => (keys[e.key.toLowerCase()] = false));

// حالة اللاعب
let health = 100;
let food = 100;

function updateStatus() {
  document.getElementById("health").textContent = Math.floor(health);
  document.getElementById("food").textContent = Math.floor(food);
}

function animate() {
  requestAnimationFrame(animate);

  let speed = 0.1;
  if (keys["w"]) player.position.z -= speed;
  if (keys["s"]) player.position.z += speed;
  if (keys["a"]) player.position.x -= speed;
  if (keys["d"]) player.position.x += speed;

  food -= 0.02;
  if (food <= 0) health -= 0.1;
  if (health <= 0) {
    alert("لقد مت! أعد تشغيل الصفحة.");
    location.reload();
  }

  updateCamera();
  updateStatus();
  renderer.render(scene, camera);
}

animate();<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <title>لعبة بقاء 3D</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div id="hud">
    ❤️ <span id="health">100</span> | 🍖 <span id="food">100</span>
  </div>

  <!-- أزرار التحكم -->
  <div id="controls">
    <button id="up">↑</button>
    <div>
      <button id="left">←</button>
      <button id="attack">⚔</button>
      <button id="right">→</button>
    </div>
    <button id="down">↓</button>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
  <script src="game.js"></script>
</body>
</html>body {
  margin: 0;
  overflow: hidden;
  background: black;
  font-family: sans-serif;
  color: white;
}

#hud {
  position: absolute;
  top: 10px;
  left: 10px;
  font-size: 20px;
  background: rgba(0, 0, 0, 0.5);
  padding: 6px 12px;
  border-radius: 8px;
  z-index: 2;
}

#controls {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 2;
  text-align: center;
}

#controls button {
  width: 50px;
  height: 50px;
  font-size: 24px;
  margin: 5px;
  border: none;
  border-radius: 12px;
  background: rgba(0, 0, 0, 0.6);
  color: white;
}const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.y = 2;

const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// الأرض
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(100, 100),
  new THREE.MeshStandardMaterial({ color: 0x228b22 })
);
ground.rotation.x = -Math.PI / 2;
scene.add(ground);

// ضوء
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(5, 10, 5);
scene.add(light);

// لاعب وهمي (لا يظهر، نستخدم الكاميرا كموقع اللاعب)
const player = new THREE.Object3D();
player.position.set(0, 2, 5);
scene.add(player);

// أشجار
for (let i = 0; i < 30; i++) {
  const trunk = new THREE.Mesh(
    new THREE.CylinderGeometry(0.2, 0.5, 3),
    new THREE.MeshStandardMaterial({ color: 0x8b4513 })
  );
  const leaves = new THREE.Mesh(
    new THREE.SphereGeometry(1.5, 8, 8),
    new THREE.MeshStandardMaterial({ color: 0x006400 })
  );
  const x = Math.random() * 80 - 40;
  const z = Math.random() * 80 - 40;
  trunk.position.set(x, 1.5, z);
  leaves.position.set(x, 4, z);
  scene.add(trunk);
  scene.add(leaves);
}

// ديناصورات
const dinos = [];
for (let i = 0; i < 5; i++) {
  const dino = new THREE.Mesh(
    new THREE.BoxGeometry(2, 1, 4),
    new THREE.MeshStandardMaterial({ color: 0x964B00 })
  );
  dino.position.set(Math.random() * 50 - 25, 0.5, Math.random() * 50 - 25);
  dino.health = 3;
  scene.add(dino);
  dinos.push(dino);
}

// HUD
let health = 100;
let food = 100;

function updateStatus() {
  document.getElementById("health").textContent = Math.floor(health);
  document.getElementById("food").textContent = Math.floor(food);
}

// حركة
let keys = {};

document.addEventListener("keydown", (e) => (keys[e.key.toLowerCase()] = true));
document.addEventListener("keyup", (e) => (keys[e.key.toLowerCase()] = false));

// تحكم باللمس
document.getElementById("up").addEventListener("touchstart", () => keys["w"] = true);
document.getElementById("down").addEventListener("touchstart", () => keys["s"] = true);
document.getElementById("left").addEventListener("touchstart", () => keys["a"] = true);
document.getElementById("right").addEventListener("touchstart", () => keys["d"] = true);
document.getElementById("attack").addEventListener("touchstart", () => attack());

document.getElementById("up").addEventListener("touchend", () => keys["w"] = false);
document.getElementById("down").addEventListener("touchend", () => keys["s"] = false);
document.getElementById("left").addEventListener("touchend", () => keys["a"] = false);
document.getElementById("right").addEventListener("touchend", () => keys["d"] = false);

// الكاميرا منظور أول شخص
camera.position.set(player.position.x, player.position.y, player.position.z);
camera.rotation.order = "YXZ";

// تحريك الكاميرا بالماوس
let mouseDown = false;
document.addEventListener("mousemove", (e) => {
  if (mouseDown) {
    camera.rotation.y -= e.movementX * 0.002;
    camera.rotation.x -= e.movementY * 0.002;
  }
});
document.addEventListener("mousedown", () => mouseDown = true);
document.addEventListener("mouseup", () => mouseDown = false);

// زر ضرب
document.getElementById("attack").addEventListener("click", attack);

function attack() {
  const attackRange = 5;
  for (let i = 0; i < dinos.length; i++) {
    const dino = dinos[i];
    const dist = dino.position.distanceTo(player.position);
    if (dist < attackRange) {
      dino.health -= 1;
      dino.material.color.set(0xff0000);
      if (dino.health <= 0) {
        scene.remove(dino);
        dinos.splice(i, 1);
      }
      break;
    }
  }
}

function animate() {
  requestAnimationFrame(animate);

  const dir = new THREE.Vector3();
  camera.getWorldDirection(dir);
  dir.y = 0;
  dir.normalize();

  let right = new THREE.Vector3().crossVectors(dir, new THREE.Vector3(0, 1, 0)).normalize();

  let speed = 0.15;
  if (keys["w"]) player.position.add(dir.clone().multiplyScalar(speed));
  if (keys["s"]) player.position.add(dir.clone().multiplyScalar(-speed));
  if (keys["a"]) player.position.add(right.clone().multiplyScalar(-speed));
  if (keys["d"]) player.position.add(right.clone().multiplyScalar(speed));

  camera.position.copy(player.position);

  food -= 0.02;
  if (food <= 0) health -= 0.1;
  if (health <= 0) {
    alert("لقد مت! أعد تشغيل الصفحة.");
    location.reload();
  }

  updateStatus();
  renderer.render(scene, camera);
}

animate();<button id="pokeball">🔴</button>#controls #pokeball {
  background-color: red;
  border: 2px solid white;
}let pokeballs = 3;
let tamedDinos = [];function throwPokeball() {
  if (pokeballs <= 0) {
    alert("❌ لا تملك كرات بوكيمون.");
    return;
  }

  const range = 6;
  for (let i = 0; i < dinos.length; i++) {
    const dino = dinos[i];
    const dist = dino.position.distanceTo(player.position);
    if (dist < range) {
      pokeballs--;
      tamedDinos.push("ديناصور #" + (tamedDinos.length + 1));
      scene.remove(dino);
      dinos.splice(i, 1);
      alert("✅ أمسكت بديناصور!");
      updateInventoryUI();
      return;
    }
  }

  alert("❌ لا يوجد ديناصور قريب.");
}function updateInventoryUI() {
  document.getElementById("inventory").textContent =
    `📦 الجرد: لحم=${inventory.meat}, جلد=${inventory.hide}, فأس=${inventory.axe ? '✅' : '❌'}, كرات=${pokeballs}, مروّض=${tamedDinos.length}`;
}document.getElementById("pokeball").addEventListener("click", throwPokeball);function throwPokeball() {
  if (pokeballs <= 0) {
    alert("❌ لا تملك كرات بوكيمون.");
    return;
  }

  const range = 6;
  for (let i = 0; i < dinos.length; i++) {
    const dino = dinos[i];
    const dist = dino.position.distanceTo(player.position);
    if (dist < range) {
      if (dino.health > 1) {
        alert("⚠️ الديناصور قوي جداً! أضعفه أولاً.");
        return;
      }

      pokeballs--;
      tamedDinos.push("ديناصور #" + (tamedDinos.length + 1));
      scene.remove(dino);
      dinos.splice(i, 1);
      alert("✅ أمسكت بديناصور ضعيف!");
      updateInventoryUI();
      return;
    }
  }

  alert("❌ لا يوجد ديناصور قريب.");
}ival in the World of Dinosaurs 
