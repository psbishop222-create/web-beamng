<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Web BeamNG Drive</title>
<style>
  body { margin: 0; overflow: hidden; background: #222; }
  canvas { display: block; }
</style>
</head>
<body>

<script src="https://cdn.jsdelivr.net/npm/three@0.154.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.154.0/examples/js/controls/OrbitControls.js"></script>

<script>
let scene = new THREE.Scene();
scene.background = new THREE.Color(0x222222);

let camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
let renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

let controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableKeys = false;

// Lighting
let ambient = new THREE.AmbientLight(0x404040, 2);
scene.add(ambient);
let dirLight = new THREE.DirectionalLight(0xffffff, 1.2);
dirLight.position.set(50, 100, 50);
scene.add(dirLight);

// Ground
let groundGeo = new THREE.PlaneGeometry(200, 200);
let groundMat = new THREE.MeshPhongMaterial({color:0x555555});
let ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI/2;
ground.receiveShadow = true;
scene.add(ground);

// Obstacles
function createObstacle(x,z){
    let geo = new THREE.BoxGeometry(2,2,2);
    let mat = new THREE.MeshPhongMaterial({color:0x888888});
    let box = new THREE.Mesh(geo, mat);
    box.position.set(x,1,z);
    scene.add(box);
    return box;
}
let obstacles = [
    createObstacle(5,5),
    createObstacle(-10, -15),
    createObstacle(20,-5)
];

// Car
class Car {
    constructor(color=0xff0000){
        this.geometry = new THREE.BoxGeometry(2,1,4);
        this.material = new THREE.MeshPhongMaterial({color:color});
        this.mesh = new THREE.Mesh(this.geometry, this.material);
        this.mesh.position.y = 0.5;
        scene.add(this.mesh);
        this.speed = 0;
        this.angle = 0;
        this.damage = 0;
    }
    update(keys){
        // Movement
        if(keys['w']) this.speed += 0.05;
        if(keys['s']) this.speed -= 0.03;
        if(keys['a']) this.angle += 0.03;
        if(keys['d']) this.angle -= 0.03;
        
        // Move car
        this.mesh.position.x += Math.sin(this.angle) * this.speed;
        this.mesh.position.z += Math.cos(this.angle) * this.speed;
        this.mesh.rotation.y = this.angle;
        this.speed *= 0.95; // friction
        
        // Check collision with obstacles
        obstacles.forEach(ob => {
            let dx = this.mesh.position.x - ob.position.x;
            let dz = this.mesh.position.z - ob.position.z;
            if(Math.sqrt(dx*dx + dz*dz) < 3){
                this.speed *= -0.5; // bounce
                this.damage += 0.1;
                this.mesh.material.color.setRGB(Math.max(1-this.damage,0),0,0);
            }
        });
    }
}

let car = new Car(0xff0000);

let keys = {};
window.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
window.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

// Camera follow
function updateCamera(){
    camera.position.set(
        car.mesh.position.x + 10*Math.sin(car.angle),
        5,
        car.mesh.position.z + 10*Math.cos(car.angle)
    );
    camera.lookAt(car.mesh.position);
}

// Main loop
function animate(){
    requestAnimationFrame(animate);
    car.update(keys);
    updateCamera();
    renderer.render(scene, camera);
}

animate();
</script>
</body>
</html>
