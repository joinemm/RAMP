<template>
    <div id="app">
        <div id="wrapper">
            <div id="left" class="column">
                <div class="top-left top">
                    <img class="logo" src="./assets/ramp_logo_white.png" />
                </div>
                <div class="bottom sidebar">
                    <Tree />
                </div>
            </div>
            <div id="right" class="column">
                <div class="top-right top">file | edit | etc</div>
                <div id="scene-container" class="bottom" v-on:mousemove="onMouseMove"></div>
            </div>
        </div>
    </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css?family=Roboto+Mono');
#app {
    font-family: 'Roboto Mono', monospace;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: white;
    height: 100vh;
}
body {
    margin: 0;
    padding: 0;
}
.logo {
    max-height: 43px;
}
#wrapper {
    height: 100%;
    display: flex;
    overflow: hidden;
    box-sizing: border-box;
}
.column {
    height: 100%;
    display: flex;
    flex-direction: column;
}
#left {
    flex-shrink: 0;
    width: 300px;
    background-color: #342f55;
}
#right {
    background-color: #f3f3f3;
    width: 100%;
}

.top-right {
    display: inline-flex;
    padding: 2px;
    background-color: grey;
}
.top-left {
    background-color: #1a1737;
    padding: 5px;
}
.top {
    flex-shrink: 0;
    color: white;
}
.bottom {
    flex-grow: 1;
    overflow-y: auto;
}
#content {
    display: flex;
    flex-direction: row;
    flex: 1;
}

.scene-container {
    display: block;
    overflow: hidden;
    width: 100%;
    height: 100%;
}
.scene-container canvas {
    overflow: hidden;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}
.sidebar {
    padding: 20px;
}
</style>

<script>
import {
    BoxGeometry,
    Mesh,
    Color,
    FogExp2,
    MeshStandardMaterial,
    PerspectiveCamera,
    Scene,
    WebGLRenderer,
    AmbientLight,
    Raycaster,
    Vector2,
    PlaneGeometry,
    MeshPhongMaterial,
    DoubleSide,
    SpotLight,
    HemisphereLight,
} from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { TransformControls } from 'three/examples/jsm/controls/TransformControls.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import Grid from './objects/grid';
import Tree from './components/Tree';

export default {
    name: 'App',
    components: { Tree },
    data() {
        return {
            camera: null,
            scene: null,
            renderer: null,
            mesh: null,
            rotate: false,
            raycaster: null,
            mouse: null,
            mouseDragStart: new Vector2(0, 0),
            transformControls: null,
            selectables: [],
            controls: null,
            loader: null,
        };
    },
    methods: {
        init() {
            let container = document.getElementById('scene-container');

            this.camera = new PerspectiveCamera(
                50,
                container.offsetWidth / container.offsetHeight,
                0.1,
                1000,
            );
            this.camera.position.set(5, 5, 5);
            this.camera.lookAt(0, 0, 0);

            this.scene = new Scene();
            this.scene.background = new Color(0xeeeeee);
            this.scene.fog = new FogExp2(0x7effee, 0.005);

            this.raycaster = new Raycaster();
            this.loader = new GLTFLoader();

            const geometry = new BoxGeometry();
            const material = new MeshStandardMaterial({
                color: 0xffc0cb,
                flatShading: true,
            });
            this.mesh = new Mesh(geometry, material);
            this.mesh.position.y = 0.5;

            this.addShadow(this.mesh);
            this.scene.add(this.mesh);
            this.selectables.push(this.mesh);

            // lights
            const ambi = new AmbientLight(0x404050, 0.5);
            // const dir = new DirectionalLight(0xffffff);
            // dir.position.set(5, 5, 4);
            // dir.target.position.set(0, 0, 0);
            // this.scene.add(dir, ambi);
            const hemi = new HemisphereLight(0xffeeb1, 0x080820, 0.5);
            const spot = new SpotLight(0xcfa98c, 1);
            spot.position.set(-10, 10, 10);
            spot.castShadow = true;
            spot.shadow.bias = -0.0001;
            spot.shadow.mapSize.width = 1024 * 4;
            spot.shadow.mapSize.height = 1024 * 4;
            this.scene.add(spot, hemi, ambi);

            const grid = new Grid(1, 10);
            grid.position.y = 0.01;
            this.scene.add(grid);

            const geometry_p = new PlaneGeometry(10, 10, 10);
            const material_p = new MeshPhongMaterial({ color: 0xdddddd, side: DoubleSide });
            const plane = new Mesh(geometry_p, material_p);
            plane.rotateX(Math.PI / 2);
            this.addShadow(plane);
            this.scene.add(plane);
            this.selectables.push(plane);

            this.renderer = new WebGLRenderer({ antialias: true });
            this.renderer.setSize(container.offsetWidth, container.offsetHeight);
            this.renderer.shadowMap.enabled = true;
            container.appendChild(this.renderer.domElement);

            // Set up controls
            this.controls = new OrbitControls(this.camera, this.renderer.domElement);
            this.controls.enableDamping = true;
            this.controls.dampingFactor = 0.1;
            this.controls.enablePan = true;
            this.controls.minDistance = 4;
            this.controls.maxDistance = 100;
            this.controls.update();

            // transform gizmos
            this.transformControls = new TransformControls(this.camera, this.renderer.domElement);
            // dont rotate camera when dragging gizmo
            this.transformControls.addEventListener(
                'dragging-changed',
                this.toggleControl.bind(this),
            );
            this.scene.add(this.transformControls);

            this.spawn('models/glTF/wooden_crate.glb');
            this.spawn('models/glTF/wooden_chair.glb');

            // window events
            window.addEventListener('resize', this.resizeCanvasToDisplaySize, false);
            window.addEventListener('pointerdown', this.onMouseDown);
            window.addEventListener('pointerup', this.onMouseUp);
        },
        addShadow(n) {
            if (n.isMesh) {
                n.castShadow = true;
                n.receiveShadow = true;
                if (n.material.map) n.material.map.anisotropy = 16;
            }
        },
        toggleControl(event) {
            this.controls.enabled = !event.value;
        },
        animate() {
            this.controls.update();
            requestAnimationFrame(this.animate);

            // find intersections

            if (this.mouse != null) {
                this.raycaster.setFromCamera(this.mouse, this.camera);
                const intersects = this.raycaster.intersectObjects(this.selectables);

                if (intersects.length > 0) {
                    if (this.intersect != intersects[0].object) {
                        if (this.intersect)
                            this.intersect.material.emissive.setHex(this.intersect.currentHex);

                        this.intersect = intersects[0].object;
                        this.intersect.currentHex = this.intersect.material.emissive.getHex();
                        this.intersect.material.emissive.setHex(0x0a0a0a);
                    }
                } else {
                    if (this.intersect)
                        this.intersect.material.emissive.setHex(this.intersect.currentHex);

                    this.intersect = null;
                }
            }

            if (this.rotate) {
                this.mesh.rotation.x += 0.01;
                this.mesh.rotation.y += 0.02;
            }
            this.renderer.render(this.scene, this.camera);
        },
        resizeCanvasToDisplaySize() {
            const canvas = this.renderer.domElement;
            const width = canvas.parentElement.clientWidth;
            const height = canvas.parentElement.clientHeight;
            this.renderer.setSize(width, height);
            this.camera.aspect = width / height;
            this.camera.updateProjectionMatrix();
        },
        // mouse position
        onMouseMove(event) {
            event.preventDefault();
            if (this.mouse == null) {
                this.mouse = new Vector2();
            }

            this.mouse.x = (event.offsetX / this.renderer.domElement.width) * 2 - 1;
            this.mouse.y = (-event.offsetY / this.renderer.domElement.height) * 2 + 1;
        },
        select(object) {
            this.transformControls.attach(object);
        },
        deselect() {
            this.transformControls.detach();
        },
        onMouseDown() {
            if (this.mouse != null) {
                this.mouseDragStart = new Vector2(this.mouse.x, this.mouse.y);
            }
        },
        onMouseUp(event) {
            // refresh moouse location my calling move event manually
            this.onMouseMove(event);
            // calculate mouse delta to determine if mouse was dragged or not
            const mouseDelta = new Vector2(
                this.mouseDragStart.x - this.mouse.x,
                this.mouseDragStart.y - this.mouse.y,
            );
            const deadzone = 0;
            if (Math.abs(mouseDelta.x) <= deadzone && Math.abs(mouseDelta.y) <= deadzone) {
                // only do selection and deselection if mouse was not dragged
                // this is to keep orbitcontrol functionality without interfering with selections
                if (this.intersect != null) {
                    this.select(this.intersect);
                } else {
                    this.deselect();
                }
            }
        },
        // eslint-disable-next-line no-unused-vars
        spawn(path) {
            this.loader.load(path, (gltf) => {
                // const scene = this.scene;
                // const selectables = this.selectables;
                gltf.scene.traverse(
                    function(child) {
                        if (child.isMesh) {
                            this.scene.add(child);
                            this.selectables.push(child);
                            this.addShadow(child);
                        }
                    }.bind(this),
                );
            });
        },
    },
    mounted() {
        this.init();
        this.animate();
    },
};
</script>
