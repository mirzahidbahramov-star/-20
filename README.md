<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Voxel World</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
    #hud {
      position: fixed; bottom: 10px; left: 10px;
      color: white; font-family: Arial; font-size: 14px;
      background: rgba(0,0,0,0.3); padding: 8px; border-radius: 6px;
    }
    #touch {
      position: fixed; right: 10px; bottom: 10px;
      display: flex; gap: 10px;
    }
    .btn {
      width: 60px; height: 60px; border-radius: 10px;
      background: rgba(255,255,255,0.25); border: 1px solid rgba(255,255,255,0.4);
      color: white; font-size: 18px; text-align: center; line-height: 60px;
      user-select: none;
    }
  </style>
</head>
<body>
  <div id="hud">Tap to move</div>
  <div id="touch">
    <div class="btn" id="left">◀</div>
    <div class="btn" id="right">▶</div>
    <div class="btn" id="jump">▲</div>
  </div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r152/three.min.js"></script>
  <script>
    // Basic scene
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // Light
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10, 20, 10);
    scene.add(light);
    scene.add(new THREE.AmbientLight(0x888888));

    // Textures
    const loader = new THREE.TextureLoader();
    const grass = loader.load('https://i.imgur.com/8Q8QK0P.png');
    const dirt = loader.load('https://i.imgur.com/Z4P0J7U.png');
    const stone = loader.load('https://i.imgur.com/wcZ8dCj.png');
    const wood = loader.load('https://i.imgur.com/JXjzJ6q.png');
    const leaves = loader.load('https://i.imgur.com/jfQ7j7L.png');
    const waterTex = loader.load('https://i.imgur.com/vwXk1Kd.png');

    const blockSize = 1;
    const blocks = [];

    function createBlock(x,y,z,texture){
      const geo = new THREE.BoxGeometry(blockSize, blockSize, blockSize);
      const mat = new THREE.MeshStandardMaterial({ map: texture });
      const mesh = new THREE.Mesh(geo, mat);
      mesh.position.set(x,y,z);
      scene.add(mesh);
      blocks.push(mesh);
    }

    // Terrain generation
    function generateWorld(){
      const size = 30;
      for(let x=-size; x<=size; x++){
        for(let z=-size; z<=size; z++){
          const height = Math.floor(2 + Math.sin(x/4)*2 + Math.cos(z/4)*2 + Math.random()*2);
          for(let y=0; y<=height; y++){
            if(y === height) createBlock(x,y,z,grass);
            else if(y > height-3) createBlock(x,y,z,dirt);
            else createBlock(x,y,z,stone);
          }
          // lake
          if(Math.abs(x) < 5 && Math.abs(z) < 5){
            for(let y=0; y<1; y++){
              createBlock(x,y,z,waterTex);
            }
          }
        }
      }
    }

    // Trees
    function createTree(x,z){
      const h = 4 + Math.floor(Math.random()*2);
      for(let y=1; y<=h; y++){
        createBlock(x,y,z,wood);
      }
      for(let dx=-2; dx<=2; dx++){
        for(let dz=-2; dz<=2; dz++){
          if(Math.abs(dx)+Math.abs(dz) < 4){
            createBlock(x+dx,h+1,z+dz,leaves);
          }
        }
      }
    }

    generateWorld();
    for(let i=0; i<15; i++){
      createTree(Math.floor(Math.random()*20-10), Math.floor(Math.random()*20-10));
    }

    // Player
    const player = new THREE.Object3D();
    player.position.set(0, 10, 0);
    scene.add(player);
    camera.position.set(0, 10, 5);
    camera.lookAt(player.position);

    // Movement
    let moveLeft=false, moveRight=false, jump=false;
    let velY=0;
    const gravity = -0.05;

    document.getElementById('left').addEventListener('touchstart', ()=>moveLeft=true);
    document.getElementById('left').addEventListener('touchend', ()=>moveLeft=false);
    document.getElementById('right').addEventListener('touchstart', ()=>moveRight=true);
    document.getElementById('right').addEventListener('touchend', ()=>moveRight=false);
    document.getElementById('jump').addEventListener('touchstart', ()=>jump=true);
    document.getElementById('jump').addEventListener('touchend', ()=>jump=false);

    function animate(){
      requestAnimationFrame(animate);

      if(moveLeft) player.position.x -= 0.1;
      if(moveRight) player.position.x += 0.1;
      if(jump && player.position.y <= 10.1) velY = 0.8;

      velY += gravity;
      player.position.y += velY;
      if(player.position.y < 10) { player.position.y = 10; velY = 0; }

      camera.position.set(player.position.x, player.position.y+2, player.position.z+6);
      camera.lookAt(player.position);

      renderer.render(scene, camera);
    }
    animate();

    window.addEventListener('resize', ()=>{
      camera.aspect = window.innerWidth/window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
