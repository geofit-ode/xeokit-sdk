<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <meta content="width=device-width, initial-scale=1" name="viewport">
    <title>XKTLoaderPlugin - Loading an IFC Model from the File System</title>
    <link href="css/styles.css" rel="stylesheet" type="text/css"/>

    <style>

        #myCanvas {
            background: #ffffff;
        }

        #myNavCubeCanvas {
            position: absolute;
            width: 250px;
            height: 250px;
            bottom: 50px;
            right: 10px;
            z-index: 200000;
        }

        /* ----------------------------------------------------------------------------------------------------------*/
        /* TreeViewPlugin */
        /* ----------------------------------------------------------------------------------------------------------*/

        #treeViewContainer {
            padding-top: 25px;
            pointer-events: all;
            height: 100%;
            overflow-y: scroll;
            overflow-x: hidden;
            position: absolute;
            background-color: rgba(255, 255, 255, 0.86);
            color: black;
            top: 0;
            z-index: 200000;
            float: left;
            left: 0;
            font-family: 'Roboto', sans-serif;
            font-size: 15px;
            user-select: none;
            -ms-user-select: none;
            -moz-user-select: none;
            -webkit-user-select: none;
            width: 550px;
        }

        #treeViewContainer ul {
            list-style: none;
            padding-left: 1.75em;
        }

        #treeViewContainer ul li {
            position: relative;
            width: 500px;
        }

        #treeViewContainer ul li a {
            background-color: #eee;
            border-radius: 50%;
            color: #000;
            display: inline-block;
            height: 1.5em;
            left: -1.5em;
            position: absolute;
            text-align: center;
            text-decoration: none;
            width: 1.5em;
        }

        #treeViewContainer ul li a.plus {
            background-color: #ded;
        }

        #treeViewContainer ul li a.minus {
            background-color: #eee;
        }

        #treeViewContainer ul li a:active {
            top: 1px;
        }

        #treeViewContainer ul li span {
            display: inline-block;
            width: calc(100% - 50px);
        }

        #treeViewContainer .highlighted-node { /* Appearance of node highlighted with TreeViewPlugin#showNode() */
            border: black solid 1px;
            background: yellow;
            color: black;
            padding-left: 1px;
            padding-right: 5px;
        }

    </style>

</head>
<body>

<canvas id="myCanvas"></canvas>

<canvas id="myNavCubeCanvas"></canvas>

<div id="treeViewContainer"></div>

<div id="info">
    <h1>XKTLoaderPlugin - Loading an IFC Model from the File System</h1><br>
    <ul>
        <li>
            <div id="time">Loading JavaScript modules...</div>
        </li>
        <li>
            <a href="./../docs/class/src/viewer/Viewer.js~Viewer.html"
               target="_other">Viewer</a>
        </li>
        <li>
            <a href="./../docs/class/src/plugins/XKTLoaderPlugin/XKTLoaderPlugin.js~XKTLoaderPlugin.html"
               target="_other">XKTLoaderPlugin</a>
        </li>
        <li>
            <a href="./../docs/class/src/plugins/TreeViewPlugin/TreeViewPlugin.js~TreeViewPlugin.html"
               target="_other">TreeViewPlugin</a>
        </li>
        <li>
            <a href="./../docs/class/src/plugins/NavCubePlugin/NavCubePlugin.js~NavCubePlugin.html"
               target="_other">NavCubePlugin</a>
        </li>
        <li>
            <a href="http://openifcmodel.cs.auckland.ac.nz/Model/Details/274"
               target="_other">Model source</a>
        </li>

    </ul>
</div>

</body>

<script type="module">

    //------------------------------------------------------------------------------------------------------------------
    // Import the modules we need for this example
    //------------------------------------------------------------------------------------------------------------------
    
    import {Viewer} from "./../src/viewer/Viewer.js";
    import {XKTLoaderPlugin} from "./../XKTLoaderPlugin/XKTLoaderPlugin.js";
    import {AmbientLight} from './../src/viewer/scene/lights/AmbientLight.js';
    import {NavCubePlugin} from "./../src/plugins/NavCubePlugin/NavCubePlugin.js";
    import {TreeViewPlugin} from "./../src/plugins/TreeViewPlugin/TreeViewPlugin.js";
    
    //------------------------------------------------------------------------------------------------------------------
    // Create a Viewer, arrange the camera, tweak x-ray and highlight materials
    //------------------------------------------------------------------------------------------------------------------

    const viewer = new Viewer({
        canvasId: "myCanvas",
        transparent: true
    });

    viewer.cameraControl.followPointer = true;

    viewer.camera.eye = [-3.933, 2.855, 27.018];
    viewer.camera.look = [4.400, 3.724, 8.899];
    viewer.camera.up = [-0.018, 0.999, 0.039];

    viewer.cameraFlight.fitFOV = 15;

    viewer.scene.xrayMaterial.fillAlpha = 0.1;
    viewer.scene.xrayMaterial.fillColor = [0, 0, 0];
    viewer.scene.xrayMaterial.edgeAlpha = 0.4;
    viewer.scene.xrayMaterial.edgeColor = [0, 0, 0];

    viewer.scene.highlightMaterial.fillAlpha = 0.3;
    viewer.scene.highlightMaterial.edgeColor = [1, 1, 0];

    //------------------------------------------------------------------------------------------------------------------
    // Create a NavCube
    //------------------------------------------------------------------------------------------------------------------

    new NavCubePlugin(viewer, {
        canvasId: "myNavCubeCanvas",
        visible: true,
        size: 250,
        alignment: "bottomRight",
        bottomMargin: 100,
        rightMargin: 10
    });

    //------------------------------------------------------------------------------------------------------------------
    // Create an IFC structure tree view
    //------------------------------------------------------------------------------------------------------------------

    const treeView = new TreeViewPlugin(viewer, {
        containerElement: document.getElementById("treeViewContainer"),
        autoExpandDepth: 1 // Initially expand the root tree node
    });

    //------------------------------------------------------------------------------------------------------------------
    // Load model and metadata
    //------------------------------------------------------------------------------------------------------------------

    const xktLoader = new XKTLoaderPlugin(viewer);

    const model = xktLoader.load({
        id: "myModel",
        src: "./../duplex/duplex.xkt",
        metaModelSrc: "./../duplex/duplex.json", // Creates a MetaObject instances in scene.metaScene.metaObjects
        excludeTypes: ["IfcSpace"],
        edges: true
    });

    const t0 = performance.now();
    document.getElementById("time").innerHTML = "Loading model...";
    model.on("loaded", function () {
        const t1 = performance.now();
        document.getElementById("time").innerHTML = "Model loaded in " + Math.floor(t1 - t0) / 1000.0 + " seconds<br>Objects: " + model.numEntities;
    });

    //------------------------------------------------------------------------------------------------------------------
    // Mouse over Entities to highlight them
    //------------------------------------------------------------------------------------------------------------------

    var lastEntity = null;

    viewer.scene.input.on("mousemove", function (coords) {

        var hit = viewer.scene.pick({
            canvasPos: coords
        });

        if (hit) {

            if (!lastEntity || hit.entity.id !== lastEntity.id) {

                if (lastEntity) {
                    lastEntity.highlighted = false;
                }

                lastEntity = hit.entity;
                hit.entity.highlighted = true;
            }
        } else {

            if (lastEntity) {
                lastEntity.highlighted = false;
                lastEntity = null;
            }
        }
    });


</script>
</html>
