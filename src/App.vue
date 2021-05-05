<template>
    <div id="app">
        <div id="wrapper">
            <div id="left" class="column">
                <div class="top-left top">
                    <img class="logo" src="./assets/ramp_logo_white.png" />
                </div>
                <div class="bottom sidebar">
                    <Tabs>
                        <Tab name="Scene" :selected="true">
                            <Tree v-bind:scene.sync="scene" />
                            <!-- hardcoded in termporarily -->
                            <div class="wrapper">
                                <h3>Robot arm</h3>
                                <div v-if="transformControls != null">
                                    <p>Section 3</p>
                                    <button
                                        @click="
                                            structure.joints[0].select(transformControls);
                                            transformControls.setMode('rotate');
                                        "
                                    >
                                        select
                                    </button>
                                </div>

                                <div v-if="transformControls != null">
                                    <p>Section 2</p>
                                    <button
                                        @click="
                                            structure.joints[1].select(transformControls);
                                            transformControls.setMode('rotate');
                                        "
                                    >
                                        select
                                    </button>
                                </div>

                                <div v-if="transformControls != null">
                                    <p>Section 1</p>
                                    <button
                                        @click="
                                            structure.joints[2].select(transformControls);
                                            transformControls.setMode('rotate');
                                        "
                                    >
                                        select
                                    </button>
                                </div>
                            </div>
                        </Tab>
                        <Tab name="Selection">
                            <ObjectView v-bind:object.sync="selected" />
                        </Tab>
                    </Tabs>
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
    max-height: 50px;
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
    padding: 10px;
}
.wrapper {
    padding: 10px;
}
</style>

<script>
import {
    BoxGeometry,
    Mesh,
    Color,
    FogExp2,
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
    Box3,
    Vector3,
    BoxHelper,
    AxesHelper,
    CylinderGeometry,
    SphereGeometry,
    Group,
    Math as THREEMath,
} from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { TransformControls } from 'three/examples/jsm/controls/TransformControls.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import Grid from './objects/grid';
import Tree from './components/Tree';
import ObjectView from './components/ObjectView';
import Tabs from './components/Tabs';
import Tab from './components/Tab';

class Part {
    // construct me from the given Mesh Factory
    // the constructor does not set size and mesh yet - use init(mesh) - later to do so
    constructor(meshFactory) {
        this.meshFactory = meshFactory;
        this.mesh = null;
        this.size = null;
        this.debug = false;
    }

    // initialize me with the given Mesh
    // calulate bounding box and size
    init(mesh) {
        this.mesh = mesh;
        this.name = mesh.name;
        var bbox = new Box3().setFromObject(this.mesh);
        this.bbox = bbox;
        this.size = new Vector3(
            bbox.max.x - bbox.min.x,
            bbox.max.y - bbox.min.y,
            bbox.max.z - bbox.max.z,
        );
        if (this.debug)
            console.log(
                'bounding box for ' +
                    this.name +
                    '=' +
                    JSON.stringify(bbox.min) +
                    JSON.stringify(bbox.max),
            );
    }

    // add my bounding box wire to the given mesh
    addBoundingBoxWire(toMesh) {
        var boxwire = new BoxHelper(this.mesh, 0xff8000);
        this.boxwire;
        boxwire.update();
        toMesh.add(boxwire);
    }

    // add an AxesHelper with my size to the given mesh
    addAxesHelper(toMesh) {
        var axis = new AxesHelper(this.size.length());
        toMesh.add(axis);
    }

    // add may bounding box wire and AxesHelper to the given mesh
    addBoxWireAndAxesHelper(toMesh) {
        this.addBoundingBoxWire(toMesh);
        this.addAxesHelper(toMesh);
    }
}

// Factory for Meshes with common properties e.g. same material / segments
class MeshFactory {
    // create me with the given default material and default segments
    constructor(material, segments) {
        this.material = material;
        this.segments = segments;
    }

    // creates a cylinder with given radius and height
    createCylinder(radius, height, cloneMaterial = false) {
        var cylinderGeometry = new CylinderGeometry(
            radius,
            radius,
            height,
            this.segments,
            this.segments,
        );
        return this.createMesh(cylinderGeometry, cloneMaterial);
    }

    // creates a sphere with given radius and height
    createSphere(radius, cloneMaterial = false) {
        var sphereGeometry = new SphereGeometry(radius, this.segments, this.segments);
        return this.createMesh(sphereGeometry, cloneMaterial);
    }

    // creates a mesh with the given geometry, optionally cloning Material
    createMesh(geometry, cloneMaterial = false) {
        var mesh = new Mesh(geometry, this.getMaterial(cloneMaterial));
        return mesh;
    }

    // get the material of this factory - when cloned is true clone a copy of the material
    // for further modification e.g. changing of color
    getMaterial(cloned) {
        var material = this.material;
        if (cloned) material = material.clone();
        return material;
    }
}

// a Joint to be used as a Pivot Point for rotation
class Joint extends Part {
    // create me from the given mesh
    // position me at the given x,y,z position
    // show pivot sphere with given pivotr radius
    constructor(meshFactory, mesh, x, y, z, pivotr) {
        // initialize my attributes
        super(meshFactory);
        this.options = null;
        this.pivotr = pivotr;

        // construct the general Part details from the given mesh
        super.init(mesh);

        // create the pivot to rotate around/about
        this.pivot = new Group();
        this.pivot.add(this.mesh);
        // shift the pivot position to fit my size + the size of the joint
        this.pivot.position.set(x, y + this.size.y / 2 + this.pivotr, z + this.size.z / 2);
        // reposition the mesh accordingly
        this.mesh.position.set(0, this.size.y / 2, 0);

        // show axes and bounding box wire frame for debugging
        this.addBoxWireAndAxesHelper(this.pivot);

        // show the pivotJoint
        this.pivotJoint = this.createPivotJoint();
        this.pivot.add(this.pivotJoint);
        // this.shiftPivot();
    }

    add_to_scene(scene) {
        scene.add(this.pivot);
    }

    select(control) {
        control.attach(this.pivot);
    }

    createPivotJoint() {
        var pivot = this.meshFactory.createCylinder(this.pivotr, this.pivotr * 2, true);
        pivot = this.meshFactory.createSphere(this.pivotr, true);
        pivot.rotation.z = THREEMath.degToRad(90);
        pivot.material.color.set('red');
        return pivot;
    }

    rotate() {
        var r = this.pivot.rotation;
        r.x += 0.001;
        r.y += 0.001;
        r.z += 0.001;
    }
} // Joint

// a structure consisting of multiple Parts
class Structure {
    // construct me from the given options
    constructor() {
        this.joints = [];
    }

    addJoint(joint) {
        this.joints.push(joint);
    }

    update() {
        for (var joint in this.joints) {
            this.joints[joint].rotate();
        }
    }
}

class Arm extends Part {
    // construct me from the given meshFactory with the given name, radius and height
    constructor(scene, meshFactory, name, radius, height) {
        super(meshFactory);
        this.radius = radius;
        this.height = height;
        this.scene = scene;
        this.cylinder = meshFactory.createCylinder(radius, height);
        this.cylinder.name = name;
        super.init(this.cylinder);
        // attributes to be fille later
        this.joint = null;
    }

    // add my joint and position me at the given x,y,z
    addJoint(x, y, z) {
        this.joint = new Joint(this.meshFactory, this.cylinder, x, y, z, this.radius);
        this.joint.add_to_scene(this.scene);
    }
}

function addArms(scene, meshFactory, structure, count) {
    var radius = 0.1;
    var height = 0.4;

    for (var i = 1; i <= count; i++) {
        var arm = new Arm(scene, meshFactory, 'arm' + i, radius, height);
        var x = 0;
        var z = 0;
        var y = height / 2;
        arm.addJoint(x, y, z);
        structure.addJoint(arm.joint);
        if (i > 1) {
            var previousJoint = structure.joints[i - 2];
            arm.joint.pivot.add(previousJoint.pivot);
        }
    }
}

export default {
    name: 'App',
    components: { ObjectView, Tabs, Tab, Tree },
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
            selected: null,
            structure: new Structure(),
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
            var material = new MeshPhongMaterial({
                color: 0x0033ff,
                specular: 0x555555,
                shininess: 200,
            });
            this.camera.position.set(5, 5, 5);
            this.camera.lookAt(0, 0, 0);

            this.scene = new Scene();
            this.scene.background = new Color(0xeeeeee);
            this.scene.fog = new FogExp2(0x7effee, 0.005);

            this.raycaster = new Raycaster();
            this.loader = new GLTFLoader();

            const geometry = new BoxGeometry();
            this.mesh = new Mesh(geometry, material);
            this.mesh.position.y = 0.5;

            this.addShadow(this.mesh);
            this.scene.add(this.mesh);
            this.mesh.position.set(-3, 0, 3);
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
            plane.name = 'Floor';
            plane.selectable = true;
            this.addShadow(plane);
            //this.scene.add(plane);
            //this.selectables.push(plane);

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

            this.spawn('models/glTF/wooden_crate.glb', new Vector3(3, 0, -3));
            this.spawn('models/glTF/wooden_chair.glb', new Vector3(-3, 0, -3));

            // window events
            window.addEventListener('resize', this.resizeCanvasToDisplaySize, false);
            container.addEventListener('pointerdown', this.onMouseDown);
            container.addEventListener('pointerup', this.onMouseUp);

            // joints stuff
            // default material to be used in MeshFactory

            var meshFactory = new MeshFactory(material, 64);
            addArms(this.scene, meshFactory, this.structure, 3);
            this.scene.add(new AxesHelper(1.5));

            //this.structure.joints[0].select(this.transformControls);

            // end joints stuff
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
            //this.structure.update();
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
            this.selected = object;
            this.transformControls.attach(object);
            this.transformControls.setMode('translate');
        },
        deselect() {
            this.selected = null;
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
        spawn(path, location = new Vector3(0, 0, 0)) {
            this.loader.load(path, (gltf) => {
                // const scene = this.scene;
                // const selectables = this.selectables;
                gltf.scene.traverse(
                    function(child) {
                        if (child.isMesh) {
                            this.scene.add(child);
                            child.position = location;
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
