<template>
    <div id="ARScene"></div>
  </template>
  <script lang="ts" setup>
  import * as THREE from 'three';
  import { ArToolkitProfile, ArToolkitSource, ArToolkitContext, ArMarkerControls } from 'ar-js-org/three.js/build/ar-threex.js';
  
  onMounted(() => {
    ArToolkitContext.baseURL = './';
  
    // init renderer
    const renderer = new THREE.WebGLRenderer({
      antialias: true,
      alpha: true,
    });
  
    renderer.setClearColor(new THREE.Color('lightgrey'), 0);
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.domElement.style.position = 'absolute';
    renderer.domElement.style.top = '0px';
    renderer.domElement.style.left = '0px';
    document.body.appendChild(renderer.domElement);
  
    // array of functions for the rendering loop
    const onRenderFcts = [] as any[];
  
    // init scene and camera
    const scene = new THREE.Scene();
  
    // Initialize a basic camera
    const camera = new THREE.Camera();
    scene.add(camera);
  
    const artoolkitProfile = new ArToolkitProfile();
    artoolkitProfile.sourceWebcam();
  
    const additionalParameters = {
      deviceId: null,
      sourceWidth: 640,
      sourceHeight: 480,
      displayWidth: 640,
      displayHeight: 480,
    };
  
    Object.assign(artoolkitProfile.sourceParameters, additionalParameters);
  
    const arToolkitSource = new ArToolkitSource(artoolkitProfile.sourceParameters);
  
    arToolkitSource.init(() => {
      onResize();
    });
  
    window.addEventListener('resize', () => {
      onResize();
    });
  
    function onResize() {
      arToolkitSource.onResizeElement();
      arToolkitSource.copyElementSizeTo(renderer.domElement);
      if (arToolkitContext.arController !== null) {
        arToolkitSource.copyElementSizeTo(arToolkitContext.arController.canvas);
      }
    }
  
    const arToolkitContext = new ArToolkitContext({
      debug: false,
      cameraParametersUrl: '@/assets/camera/data.dat',
      detectionMode: 'mono',
      canvasWidth: 640,
      canvasHeight: 490,
      imageSmoothingEnabled: true,
    });
  
    arToolkitContext.init(() => {
      camera.projectionMatrix.copy(arToolkitContext.getProjectionMatrix());
    });
  
    onRenderFcts.push(() => {
      if (arToolkitSource.ready === false) return;
      arToolkitContext.update(arToolkitSource.domElement);
    });
  
    // First marker
    const markerGroup1 = new THREE.Group();
    
    scene.add(markerGroup1);
  
    new ArMarkerControls(arToolkitContext, markerGroup1, {
      type: 'pattern',
      patternUrl: '@/assets/camera/patt.hiro',
      smooth: true,
      smoothCount: 5,
      smoothTolerance: 0.01,
      smoothThreshold: 2,
    });
  
    const markerScene1 = new THREE.Scene();

    markerGroup1.add(markerScene1);
  
    const mesh1 = new THREE.Mesh(
      new THREE.BoxGeometry(1, 1, 1),
      new THREE.MeshNormalMaterial({
        transparent: true,
        opacity: 0.5,
        side: THREE.DoubleSide,
      })
    );

    mesh1.position.y = 0.5;

    markerScene1.add(mesh1);
  
    // Second marker
    const markerGroup2 = new THREE.Group();

    scene.add(markerGroup2);
  
    new ArMarkerControls(arToolkitContext, markerGroup2, {
      type: 'pattern',
      patternUrl: '@/assets/camera/pattern-market.patt', // New pattern URL
      smooth: true,
      smoothCount: 5,
      smoothTolerance: 0.01,
      smoothThreshold: 2,
    });
  
    const markerScene2 = new THREE.Scene();

    markerGroup2.add(markerScene2);
  
    const mesh2 = new THREE.Mesh(
      new THREE.TorusKnotGeometry(0.3, 0.1, 64, 16),
      new THREE.MeshNormalMaterial()
    );

    mesh2.position.y = 0.5;

    markerScene2.add(mesh2);
  
    onRenderFcts.push((delta: any) => {
      mesh1.rotation.x += delta * Math.PI;
      mesh2.rotation.y += delta * Math.PI;
    });
  
    onRenderFcts.push(() => {
      renderer.render(scene, camera);
    });
  
    let lastTimeMsec = null as number | null;
    requestAnimationFrame(function animate(nowMsec) {

      requestAnimationFrame(animate);

      lastTimeMsec = lastTimeMsec || nowMsec - 1000 / 60;

      const deltaMsec = Math.min(200, nowMsec - lastTimeMsec);

      lastTimeMsec = nowMsec;

      onRenderFcts.forEach((onRenderFct) => {
        onRenderFct(deltaMsec / 1000, nowMsec / 1000);
      })

    });
  });
  </script>
  