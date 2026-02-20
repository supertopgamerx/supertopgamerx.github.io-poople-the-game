<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Poople 3D Adventure</title>
<style>
    body {
        margin: 0;
        overflow: hidden;
        font-family: Arial, sans-serif;
        background: linear-gradient(to top, #87ceeb, #ffffff);
    }

    #ui {
        position: absolute;
        top: 15px;
        left: 15px;
        background: rgba(0,0,0,0.6);
        color: white;
        padding: 10px 15px;
        border-radius: 12px;
        font-size: 14px;
    }
</style>
</head>
<body>

<div id="ui">
🎮 Poople 3D Adventure<br>
Move: WASD<br>
Jump: Space
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>
let scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

// Camera
let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
camera.position.set(0, 3, 6);

// Renderer
let renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

// Lighting
let light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(5, 10, 7);
light.castShadow = true;
scene.add(light);

let ambient = new THREE.AmbientLight(0xffffff, 0.4);
scene.add(ambient);

// Ground
let groundGeo = new THREE.PlaneGeometry(100, 100);
let groundMat = new THREE.MeshStandardMaterial({ color: 0x55aa55 });
let ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

// Poople Body (Cone stack shape)
let poopleGroup = new THREE.Group();

function createLayer(radius, height, yPos) {
    let geo = new THREE.ConeGeometry(radius, height, 32);
    let mat = new THREE.MeshStandardMaterial({ color: 0x8b4513 });
    let layer = new THREE.Mesh(geo, mat);
    layer.position.y = yPos;
    layer.castShadow = true;
    return layer;
}

poopleGroup.add(createLayer(1.5, 1.5, 0.75));
poopleGroup.add(createLayer(1.1, 1.3, 1.7));
poopleGroup.add(createLayer(0.8, 1.0, 2.5));

// Emoji Face (canvas texture)
let canvas = document.createElement("canvas");
canvas.width = 256;
canvas.height = 256;
let ctx = canvas.getContext("2d");

ctx.fillStyle = "#8b4513";
ctx.fillRect(0, 0, 256, 256);
ctx.font = "180px Arial";
ctx.textAlign = "center";
ctx.textBaseline = "middle";
ctx.fillText("💩", 128, 128);

let texture = new THREE.CanvasTexture(canvas);
let faceGeo = new THREE.PlaneGeometry(1.5, 1.5);
let faceMat = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
let face = new THREE.Mesh(faceGeo, faceMat);
face.position.set(0, 2.3, 1.2);

poopleGroup.add(face);
scene.add(poopleGroup);

// Movement
let keys = {};
let velocityY = 0;
let isOnGround = true;

document.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
document.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

function animate() {
    requestAnimationFrame(animate);

    // Movement
    if (keys["w"]) poopleGroup.position.z -= 0.1;
    if (keys["s"]) poopleGroup.position.z += 0.1;
    if (keys["a"]) poopleGroup.position.x -= 0.1;
    if (keys["d"]) poopleGroup.position.x += 0.1;

    // Jump
    if (keys[" "] && isOnGround) {
        velocityY = 0.2;
        isOnGround = false;
    }

    velocityY -= 0.01;
    poopleGroup.position.y += velocityY;

    if (poopleGroup.position.y <= 0) {
        poopleGroup.position.y = 0;
        velocityY = 0;
        isOnGround = true;
    }

    // Camera follow
    camera.position.x = poopleGroup.position.x;
    camera.position.z = poopleGroup.position.z + 6;
    camera.lookAt(poopleGroup.position);

    renderer.render(scene, camera);
}

animate();

// Resize Fix
window.addEventListener("resize", () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>

</body>
</html>
