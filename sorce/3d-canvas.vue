<template>
  <section class="d-flex h-100" style="max-height: 100%">
    <canvas id="renderCanvas" ref="thiscanvas" style="width: 100%" />
  </section>
</template>

<script>
import * as BABYLON from 'babylonjs'
import { GridMaterial } from 'babylonjs-materials'
import 'babylonjs-loaders'
import * as GUI from 'babylonjs-gui'

export default {
  props: {
    getItem: {
      type: Object,
      default: () => {},
    },
    gridVisible: {
      type: Boolean,
      default: false,
    },
    makeSceneAni: { type: Object, default: () => {} },
  },
  data() {
    return {
      saveHeight: 0,
      engine: null,
      canvas: null,
      canvas2: null,
      camera: null,
      camera2: null,
      scene: null,
      meshes: [],
      defaultCameraOption: {
        positionX: 0,
        positionY: 0,
        positionZ: 0,
        targetX: 0,
        targetY: 0,
        targetZ: 0,
        wheelPrecision: 20, //높아질수록 감도가 낮아짐
        lowerRadiusLimit: 2,
        upperRadiusLimit: 100,
        panningSensibility: 1000, //높아질수록 감도가 낮아짐
        fullCam: null,
        gizmoManager: null,
        localClickObj: this.clickObj,
      },
      ground: null,
      zoomLevel: 5,
      loadModel: null,
      guiTexture: null,
      selectionLayer: '',
      nowClickObjId: '',
      nowClickObj: [],
      downloadModelingMeshList: {},
      nowDownloadModelingMeshList: {},
      rect: null,

      createAniList: [],
      isDragging: false,
      startX: 0,
      startY: 0,
      positionSvg:
        'data:image/svg+xml;base64,' +
        btoa(
          '<svg viewBox="0 0 24 24" style="height: 240px; width: 240px;"><path fill="currentColor" d="M13,6V11H18V7.75L22.25,12L18,16.25V13H13V18H16.25L12,22.25L7.75,18H11V13H6V16.25L1.75,12L6,7.75V11H11V6H7.75L12,1.75L16.25,6H13Z"></path></svg>'
        ),
      trashIcon:
        'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNDIiIGhlaWdodD0iNDIiIHZpZXdCb3g9IjAgMCA0MiA0MiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPGcgaWQ9InRyYXNoIj4KPHJlY3Qgd2lkdGg9IjQyIiBoZWlnaHQ9IjQyIiByeD0iMjEiIGZpbGw9IndoaXRlIi8+CjxwYXRoIGlkPSJWZWN0b3IgODkyIiBkPSJNMTQgMTdWMjguM0MxNCAyOS43OTEyIDE1LjIwODggMzEgMTYuNyAzMUgyNS4zQzI2Ljc5MTIgMzEgMjggMjkuNzkxMiAyOCAyOC4zVjE3IiBzdHJva2U9IiMxRTFFMUUiIHN0cm9rZS13aWR0aD0iMS44IiBzdHJva2UtbGluZWNhcD0icm91bmQiLz4KPHBhdGggaWQ9IlZlY3RvciA4OTMiIGQ9Ik0xMyAxNEgyOSIgc3Ryb2tlPSIjMUUxRTFFIiBzdHJva2Utd2lkdGg9IjEuOCIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIi8+CjxyZWN0IGlkPSJSZWN0YW5nbGUgMTg0NCIgeD0iMTkiIHk9IjExIiB3aWR0aD0iNCIgaGVpZ2h0PSIzIiByeD0iMC40MDUiIGZpbGw9IiMxRTFFMUUiLz4KPHBhdGggaWQ9IlZlY3RvciA4OTQiIGQ9Ik0xOSAyMFYyNiIgc3Ryb2tlPSIjMUUxRTFFIiBzdHJva2Utd2lkdGg9IjEuOCIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIi8+CjxwYXRoIGlkPSJWZWN0b3IgODk1IiBkPSJNMjMgMjBWMjYiIHN0cm9rZT0iIzFFMUUxRSIgc3Ryb2tlLXdpZHRoPSIxLjgiIHN0cm9rZS1saW5lY2FwPSJyb3VuZCIvPgo8L2c+Cjwvc3ZnPgo=',
    }
  },
  computed: {
    cameraOption() {
      const camera = this.sceneOption?.camera ? this.sceneOption?.camera : this.defaultCameraOption
      return camera
    },
    gractorBaseUrl() {
      return this.$config.viewtrack.gractorBaseUrl
    },
  },
  watch: {
    getItem(newValue) {
      if (newValue?._id && newValue._id != this.nowClickObjId) {
        this.loadGlb(newValue)
      }
    },
    gridVisible(newVale) {
      if (newVale) {
        this.ground.isVisible = true
      } else {
        this.ground.isVisible = false
      }
    },
    downloadModelingMeshList(newVal) {
      if (Object.keys(newVal).length === 0) this.$emit('reset-click-component', null)
      this.$emit('modeling-list', this.downloadModelingMeshList)
      this.$emit('model-list-one', this.nowDownloadModelingMeshList)
    },
    makeSceneAni(newVal) {
      console.info(newVal)
      if (newVal.animationType.includes('Light')) {
        let makeLight
        if (newVal.animationType === 'pointLight') {
          makeLight = new BABYLON.PointLight('pointLight', new BABYLON.Vector3(0, 0, 0), this.scene)

          makeLight.diffuse = new BABYLON.Color3(1, 0, 0) // 빨간색 조명 설정 (diffuse)
          makeLight.specular = new BABYLON.Color3(1, 0, 0)
          makeLight.range = 100 // 빛이 영향을 미치는 최대 거리
          makeLight.radius = 5 // 빛의 크기
        } else if (newVal.animationType === 'spotLight') {
          makeLight = new BABYLON.SpotLight('spotLight', new BABYLON.Vector3(0, 10, 0), new BABYLON.Vector3(0, -1, 0), Math.PI / 3, 2, this.scene)
          makeLight.diffuse = new BABYLON.Color3(1, 1, 0) // 노란색 조명 설정
          makeLight.specular = new BABYLON.Color3(1, 1, 0) // 노란색 스펙큘러 설정
        } else if (newVal.animationType === 'directionLight') {
          makeLight = new BABYLON.DirectionalLight('directionalLight', new BABYLON.Vector3(0, -1, 0), this.scene)

          makeLight.diffuse = new BABYLON.Color3(0, 1, 0) // 녹색 조명 설정
          makeLight.specular = new BABYLON.Color3(0, 1, 0) // 녹색 스펙큘러 설정
        } else if (newVal.animationType === 'domeLight') {
          makeLight = new BABYLON.HemisphericLight('hemisphericLight', new BABYLON.Vector3(0, 1, 0), this.scene)

          makeLight.diffuse = new BABYLON.Color3(0, 0, 1) // 파란색 조명 설정
          makeLight.specular = new BABYLON.Color3(0, 0, 1) // 파란색 스펙큘러 설정
          makeLight.groundColor = new BABYLON.Color3(0, 1, 0) // 녹색 지면 조명 설정
        }
        makeLight.intensity = 100
        //console.info(makeLight, newVal)
        if (makeLight?._position) {
          makeLight._position._x = newVal.sceneAniProperties.posX
          makeLight._position._y = newVal.sceneAniProperties.posY
          makeLight._position._z = newVal.sceneAniProperties.posZ
        }
        console.info(makeLight)
        makeLight.id = newVal.sceneAniTempatedId
        this.createAniList.push(makeLight)
        // const lightGizmo = new BABYLON.LightGizmo()
        // lightGizmo.light = makeLight
        // this.gizmoManager.positionGizmoEnabled = true
        // this.gizmoManager.rotationGizmoEnabled = true
        // this.gizmoManager.scaleGizmoEnabled = true
        // this.gizmoManager.attachToNode(lightGizmo.attachedNode)
      }
    },
  },
  beforeDestroy() {
    if (this.scene) {
      this.scene.dispose()
    }
    if (this.engine) {
      this.engine.stopRenderLoop()
      this.engine.dispose()
    }
    window.removeEventListener('resize', this.handleResize)
  },
  async mounted() {
    //인스턴스 생성 후 단 한번 호출
    await this.$nextTick() //돔이 다 그려질때까지 멈춤
    this.canvas = document.getElementById('renderCanvas')
    this.engine = new BABYLON.Engine(this.canvas, true)
    this.scene = await this.createScene()
    this.engine.runRenderLoop(() => {
      this.scene.render()
    })

    const resizeObserver = new ResizeObserver(() => {
      if (this.engine) this.engine.resize()
    })
    resizeObserver.observe(this.canvas)
    this.guiTexture = GUI.AdvancedDynamicTexture.CreateFullscreenUI('UI')
    this.scene.onPointerDown = (evt, pickResult) => {
      //this.selectionLayer.removeAllMeshes()
      //this.$emit('delete-click', true)
      if (pickResult.pickedMesh == null) {
        this.allgizmofalse()
      }
      if (evt.button !== 0) return // 왼쪽 클릭만 처리
      this.guiTexture.rootContainer.children.forEach(control => {
        control.isVisible = false
      })
    }
    this.scene.onPointerObservable.add(pointerInfo => {
      switch (pointerInfo.type) {
        case BABYLON.PointerEventTypes.POINTERDOWN: {
          this.isDragging = false
          this.startX = pointerInfo.event.clientX
          this.startY = pointerInfo.event.clientY
          break
        }
        case BABYLON.PointerEventTypes.POINTERMOVE: {
          const deltaX = pointerInfo.event.clientX - this.startX
          const deltaY = pointerInfo.event.clientY - this.startY
          if (Math.sqrt(deltaX * deltaX + deltaY * deltaY) >= 1) {
            this.isDragging = true
          }
          break
        }
        case BABYLON.PointerEventTypes.POINTERUP: {
          if (!this.isDragging) {
            this.selectionLayer.removeAllMeshes()
            this.$emit('delete-click', true)
          }
          this.isDragging = false
          break
        }
      }
    })

    this.maekPositionGUI()
    this.hideGUI()
    document.addEventListener('keydown', this.mainkeydown)
    window.addEventListener('resize', this.handleResize)
    const parentElement = this.$el
    this.saveHeight = parentElement.clientHeight
    this.handleResize()
  },
  methods: {
    async createScene() {
      let scene = new BABYLON.Scene(this.engine)

      this.camera = new BABYLON.ArcRotateCamera(
        'Camera',
        0,
        Math.PI / 2,
        10,
        new BABYLON.Vector3(this.cameraOption.targetX, this.cameraOption.targetY, this.cameraOption.targetZ),
        scene
      )
      this.camera.attachControl(this.canvas, true)
      this.camera.wheelPrecision = this.cameraOption.wheelPrecision
      this.camera.lowerRadiusLimit = this.cameraOption.lowerRadiusLimit
      this.camera.upperRadiusLimit = this.cameraOption.upperRadiusLimit
      this.camera.panningSensibility = this.cameraOption.panningSensibility
      this.camera.inputs.attached.pointers.buttons = [0, 1, 2]
      this.camera.position = new BABYLON.Vector3(this.cameraOption.positionX, this.cameraOption.positionY, this.cameraOption.positionZ)

      let lighting = BABYLON.CubeTexture.CreateFromPrefilteredData('/viewtrack/source/components/cms/widget/glb-viewer/environmentSpecular.env', scene)
      lighting.name = 'runyonCanyon'
      lighting.gammaSpace = false

      scene.environmentTexture = lighting
      scene.createDefaultSkybox(null, true, (scene.activeCamera.maxZ - scene.activeCamera.minZ) / 2, 0.3, false)
      scene.createDefaultLight()
      scene.environmentTexture.intensity = 10
      scene.clearColor = new BABYLON.Color4(0, 0, 0, 0) //배경 투명화
      scene.createDefaultCameraOrLight(false, false, true) //배경 투명화

      this.selectionLayer = new BABYLON.HighlightLayer('selectionLayer', this.scene)

      this.camera2 = new BABYLON.ArcRotateCamera(
        'Camera2',
        0,
        Math.PI / 2,
        10,
        new BABYLON.Vector3(this.cameraOption.targetX, this.cameraOption.targetY, this.cameraOption.targetZ),
        scene
      )
      //new BABYLON.ArcRotateCamera('camera1', (5 * Math.PI) / 8, (5 * Math.PI) / 8, 30, new BABYLON.Vector3(0, 2, 0), scene)
      this.camera2.attachControl(this.canvas, true)

      // Two Viewports
      this.camera.viewport = new BABYLON.Viewport(0.5, 0, 0.5, 1)
      this.camera2.viewport = new BABYLON.Viewport(0, 0, 0.5, 1)

      this.canvas.addEventListener('pointerdown', event => {
        const pickInfo1 = scene.pick(scene.pointerX, scene.pointerY, undefined, false, this.camera)
        const pickInfo2 = scene.pick(scene.pointerX, scene.pointerY, undefined, false, this.camera2)
        if (scene.pointerX > this.canvas.width / 2) {
          // 첫 번째 뷰포트 내에서의 마우스 이벤트 처리
          this.camera.attachControl(this.canvas, true) // 첫 번째 카메라 반응
          this.camera2.detachControl(this.canvas) // 두 번째 카메라 반응 중지
        }

        if (scene.pointerX <= this.canvas.width / 2) {
          // 두 번째 뷰포트 내에서의 마우스 이벤트 처리
          this.camera2.attachControl(this.canvas, true) // 두 번째 카메라 반응
          this.camera.detachControl(this.canvas) // 첫 번째 카메라 반응 중지
        }
      })
      scene.activeCameras.push(this.camera)
      scene.activeCameras.push(this.camera2)
      //const sphere = MeshBuilder.CreateSphere('sphere1', { diameter: 2, segments: 32 }, scene)
      this.gizmoManager = new BABYLON.GizmoManager(scene)
      this.createGizmo()

      this.createGridMaterial(this.scene, 20) // 20x20 크기의 그리드 생성
      return scene
    },
    handleResize() {
      this.canvas.style.height = `${this.saveHeight}px`
      this.engine.resize()
    },
    handleParentClick(gettype) {
      if (gettype) {
        this.camera.radius -= this.zoomLevel
      } else {
        this.camera.radius += this.zoomLevel
      }
    },
    async loadGlb(getItem) {
      if (!this.scene) return

      try {
        const result = await BABYLON.SceneLoader.ImportMeshAsync('', 'data:', getItem.srcPath, this.scene)
        const loadedMeshes = result.meshes
        let parentMesh = loadedMeshes[0]
        await Promise.all(
          loadedMeshes.map(async mesh => {
            if (parentMesh.name.includes(this.removePrimitiveSuffix(mesh.name)) && parentMesh !== mesh.parent && mesh.name != '__root__') {
              mesh.parent = parentMesh
            } else {
              parentMesh = mesh
            }
            mesh.isPickable = true
            mesh.actionManager = new BABYLON.ActionManager(this.scene)
            mesh.actionManager.registerAction(
              new BABYLON.ExecuteCodeAction(BABYLON.ActionManager.OnLeftPickTrigger, () => {
                this.gizmoManager.attachToMesh(parentMesh)
                this.gizmoOn('position')
              })
            )
            mesh.actionManager.registerAction(
              new BABYLON.ExecuteCodeAction(BABYLON.ActionManager.OnRightPickTrigger, () => {
                this.showGUI() // 메쉬 클릭 시 GUI 표시 함수 호출
                this.nowClickObjId = getItem.createId
                this.nowClickObj = loadedMeshes
                this.updateGUIPosition(result.meshes)
              })
            )
          })
        )
        this.loadModel = result // 로드된 메쉬 저장
        this.gizmoManager.attachToMesh(parentMesh)

        let obj = {
          createId: getItem.createId,
          meshId: getItem._id,
          name: getItem.properties.name,
          mesh: this.loadModel,
        }
        this.nowDownloadModelingMeshList = obj
        this.downloadModelingMeshList = { ...this.downloadModelingMeshList, [getItem.createId]: obj }
      } catch (error) {
        console.error('에러 발생', error)
      }
    },

    generateRandomId() {
      const randomString = Math.random().toString(36).substr(2, 9) // 9자리 랜덤 문자열 생성
      return `model${randomString}`
    },
    removePrimitiveSuffix(name) {
      const suffix = '_primitive'
      const index = name.indexOf(suffix)
      if (index !== -1) {
        return name.substring(0, index)
      }
      return name
    },
    deleteAni(getId) {
      console.info(this.makeSceneAni)
      console.info(this.createAniList)
      this.createAniList.forEach(element => {
        if (element.id === getId) {
          console.info(element, this.scene)
          this.scene.lights.forEach(ele => {
            if (ele.id === getId) {
              console.info(ele)
              ele.dispose()
            }
          })
        }
      })
    },
    clearAllGlb(getId) {
      for (let key in this.downloadModelingMeshList) {
        if (key === getId) {
          const item = this.downloadModelingMeshList[key]
          // 아이템 삭제
          this.$delete(this.downloadModelingMeshList, key)

          // 메쉬 삭제
          for (let i = 0; i < item.mesh.meshes.length; i++) {
            const mesh = item.mesh.meshes[i]
            // 부모가 없는 경우 메쉬를 삭제하고 루프를 종료
            if (!mesh.parent) {
              this.deleteMeshFromScene(mesh)
              break
            }
          }
          break
        }
      }
      // for (let i = 0; i < this.nowClickObj.length; i++) {
      //   const mesh = this.nowClickObj[i]
      //   // 부모가 없는 경우 메쉬를 삭제하고 루프를 종료
      //   if (!mesh.parent) {
      //     this.deleteMeshFromScene(mesh)
      //     break
      //   }
      // }
      // if (this.loadModel?.meshes) {
      //   this.loadModel.meshes.forEach(mesh => {
      //     mesh.dispose()
      //   })

      //   this.loadModel?.materials?.forEach(material => {
      //     material.dispose()
      //   })

      //   if (this.loadModel?.skeletons) {
      //     this.loadModel.skeletons.forEach(skeleton => {
      //       skeleton.dispose()
      //     })
      //   }
      //   this.loadModel = null
      // }
    },
    deleteMeshFromScene(mesh) {
      // 메쉬가 씬에 존재하는지 확인
      if (!mesh || !this.scene) {
        console.error('Mesh 또는 Scene이 존재하지 않습니다.')
        return
      }

      // 씬에서 메쉬를 제거
      this.scene.removeMesh(mesh)
      mesh.dispose() // 메쉬를 메모리에서 제거
    },
    maekGUI() {
      if (!this.guiTexture) {
        this.guiTexture = GUI.AdvancedDynamicTexture.CreateFullscreenUI('UI')
      }
      let rect = new GUI.Rectangle()
      rect.height = '52px'
      rect.width = '104px' // 너비 설정
      rect.thickness = 0
      rect.cornerRadius = 50
      rect.background = '#fff'

      let target = new GUI.Image('target', this.trashIcon)
      target.width = '42px'
      target.height = '42px'
      target.left = '10px'
      target.hoverCursor = 'pointer'
      target.horizontalAlignment = GUI.Control.HORIZONTAL_ALIGNMENT_LEFT

      rect.addControl(target)
      let text = new GUI.TextBlock()
      text.size = '14px'
      text.text = '삭제'
      text.left = '16px'
      text.horizontalAlignment = GUI.Control.HORIZONTAL_ALIGNMENT_RIGHT

      rect.addControl(text)

      this.guiTexture.addControl(rect) // GUI 텍스처에 Rectangle 추가
      rect.shadowBlur = 4 // 그림자의 흐림 정도
      rect.shadowOffsetX = 0 // 그림자의 X축 오프셋
      rect.shadowOffsetY = 5 // 그림자의 Y축 오프셋
      rect.shadowColor = 'rgba(0, 0, 0, 0.25)' // 그림자의 색상

      rect.onPointerClickObservable.add((eventData, eventState) => {
        this.clearAllGlb(this.nowClickObjId)
        this.hideGUI()
      })
      // setTimeout(() => {
      //   this.guiTexture = GUI.AdvancedDynamicTexture.CreateFullscreenUI('UI')
      // }, 0)
      //this.$emit('reset-click-component', null)

      this.rect = rect
      // let findparnet = mesh[0]
      // while (findparnet?._parentNode !== null) {
      //   if (findparnet == null) break
      //   findparnet = findparnet._parentNode
      // }

      // let minVertex = new Vector3(Number.MAX_VALUE, Number.MAX_VALUE, Number.MAX_VALUE)
      // let maxVertex = new Vector3(Number.MIN_VALUE, Number.MIN_VALUE, Number.MIN_VALUE)
      // findparnet.getChildMeshes().forEach(mesh => {
      //   let boundingBox = mesh.getBoundingInfo()

      //   minVertex = Vector3.Minimize(minVertex, boundingBox.boundingBox.minimumWorld)
      //   maxVertex = Vector3.Maximize(maxVertex, boundingBox.boundingBox.maximumWorld)
      // })
      // let center = Vector3.Center(minVertex, maxVertex)
      // let height = maxVertex.y - minVertex.y
      // let guiPosition = new Vector3(center.x, center.y + height, center.z)
      // const newpositionsphere = MeshBuilder.CreateSphere('sphere', { diameter: 1 }, this.scene)
      // newpositionsphere.position = guiPosition
      // var material = new BABYLON.StandardMaterial('material', this.scene)
      // material.alpha = 0
      // newpositionsphere.material = material
      // rect.linkWithMesh(newpositionsphere)

      // this.showGUI(false)
    },
    maekPositionGUI() {
      if (!this.guiTexture) {
        this.guiTexture = GUI.AdvancedDynamicTexture.CreateFullscreenUI('UI')
      }
      let rect = new GUI.Rectangle()
      rect.height = '50px'
      rect.width = '238px' // 너비 설정
      rect.thickness = 0
      rect.cornerRadius = 50
      rect.background = '#fff'
      this.guiTexture.addControl(rect)

      var container = new GUI.StackPanel()
      container.width = '238px'
      container.height = '50px'
      container.horizontalAlignment = GUI.Control.HORIZONTAL_ALIGNMENT_CENTER
      container.verticalAlignment = GUI.Control.VERTICAL_ALIGNMENT_TOP
      container.top = '1px'
      container.left = '7px'

      container.background = '#fff'
      container.isVertical = false // 가로 정렬

      rect.addControl(container)
      var leftMargin = 0 // 초기 왼쪽 마진
      var marginIncrement = 1.92 // 추가할 마진 크기
      // 각 이미지 버튼 생성
      var createButton = function (id, src) {
        var button = GUI.Button.CreateImageOnlyButton(id, src)
        button.width = '42px'
        button.height = '42px'
        button.left = leftMargin + 'px'
        button.thickness = 0
        leftMargin += 42 + marginIncrement
        return button
      }
      var createSpacer = function (width) {
        var spacer = new GUI.Control()
        spacer.width = width + 'px'
        return spacer
      }

      // 버튼 추가
      let btn_scale = createButton('studio-obj-func-scale', this.trashIcon)
      let btn_rotate = createButton('studio-obj-func-rotate', this.trashIcon)
      let btn_move = createButton('studio-obj-func-move', this.trashIcon)
      let btn_lock = createButton('studio-obj-func-info', this.trashIcon)
      let btn_delete = createButton('studio-obj-func-delete', this.trashIcon)

      btn_scale.onPointerClickObservable.add(() => {
        this.gizmoManager.gizmos.positionGizmo.updateGizmoPositionToMatchAttachedMesh = false
        this.gizmoManager.gizmos.scaleGizmo.updateGizmoScaleToMatchAttachedMesh = false
        this.gizmoManager.gizmos.rotationGizmo.updateGizmoRotationToMatchAttachedMesh = false
        this.allgizmofalse()

        this.gizmoManager.scaleGizmoEnabled = true
        this.gizmoManager.gizmos.scaleGizmo.updateGizmoRotationToMatchAttachedMesh = true
      })
      btn_rotate.onPointerClickObservable.add(() => {
        this.gizmoManager.gizmos.positionGizmo.updateGizmoPositionToMatchAttachedMesh = false
        this.gizmoManager.gizmos.scaleGizmo.updateGizmoScaleToMatchAttachedMesh = false
        this.gizmoManager.gizmos.rotationGizmo.updateGizmoRotationToMatchAttachedMesh = false
        this.allgizmofalse()

        this.gizmoManager.rotationGizmoEnabled = true
        this.gizmoManager.gizmos.rotationGizmo.updateGizmoRotationToMatchAttachedMesh = false
      })
      btn_move.onPointerClickObservable.add(() => {
        this.gizmoManager.gizmos.positionGizmo.updateGizmoPositionToMatchAttachedMesh = false
        this.gizmoManager.gizmos.scaleGizmo.updateGizmoScaleToMatchAttachedMesh = false
        this.gizmoManager.gizmos.rotationGizmo.updateGizmoRotationToMatchAttachedMesh = false
        this.allgizmofalse()

        this.gizmoManager.positionGizmoEnabled = true
        this.gizmoManager.gizmos.positionGizmo.updateGizmoPositionToMatchAttachedMesh = true
      })

      btn_delete.onPointerClickObservable.add(() => {
        // this.gizmoManager._attachedMesh.dispose()
        this.clearAllGlb(this.nowClickObjId)
        this.hideGUI()
      })

      btn_lock.onPointerClickObservable.add(() => {})

      container.addControl(btn_move)
      container.addControl(createSpacer(3)) // 10px 간격
      container.addControl(btn_rotate)
      container.addControl(createSpacer(3)) // 10px 간격

      container.addControl(btn_scale)
      container.addControl(createSpacer(3)) // 10px 간격

      container.addControl(btn_lock)
      container.addControl(createSpacer(3)) // 10px 간격
      container.addControl(btn_delete)

      this.rect = rect
    },
    hideGUI() {
      if (this.rect) {
        this.rect.isVisible = false
      }
    },
    updateGUIPosition(mesh) {
      let findParent = mesh[0]
      while (findParent?._parentNode !== null) {
        findParent = findParent._parentNode
      }

      let minVertex = new BABYLON.Vector3(Number.MAX_VALUE, Number.MAX_VALUE, Number.MAX_VALUE)
      let maxVertex = new BABYLON.Vector3(Number.MIN_VALUE, Number.MIN_VALUE, Number.MIN_VALUE)
      findParent.getChildMeshes().forEach(childMesh => {
        let boundingBox = childMesh.getBoundingInfo().boundingBox

        minVertex = BABYLON.Vector3.Minimize(minVertex, boundingBox.minimumWorld)
        maxVertex = BABYLON.Vector3.Maximize(maxVertex, boundingBox.maximumWorld)
      })

      let center = BABYLON.Vector3.Center(minVertex, maxVertex)
      let height = maxVertex.y - minVertex.y
      let guiPosition = new BABYLON.Vector3(center.x, center.y + height, center.z)
      const newpositionsphere = BABYLON.MeshBuilder.CreateSphere('sphere', { diameter: 1 }, this.scene)
      newpositionsphere.position = guiPosition

      let material = new BABYLON.StandardMaterial('material', this.scene)
      material.alpha = 0
      newpositionsphere.material = material
      this.rect.linkWithMesh(newpositionsphere)
      // rect.shadowBlur = 4 // 그림자의 흐림 정도
      // rect.shadowOffsetX = 0 // 그림자의 X축 오프셋
      // rect.shadowOffsetY = 5 // 그림자의 Y축 오프셋
      // rect.shadowColor = 'rgba(0, 0, 0, 0.25)' // 그림자의 색상

      // rect.onPointerClickObservable.add((eventData, eventState) => {
      //   this.clearAllGlb()
      //   if (this.guiTexture) {
      //     this.guiTexture.dispose()
      //     this.guiTexture = null
      //   }

      //   setTimeout(() => {
      //     this.guiTexture = GUI.AdvancedDynamicTexture.CreateFullscreenUI('UI')
      //   }, 0)
      //   this.$emit('reset-click-component', null)
      // })
      this.showGUI(false)
    },
    showGUI() {
      if (this.rect) {
        this.rect.isVisible = true
      }
    },
    // showGUI(getType) {
    //   if (this.guiTexture) {
    //     this.guiTexture.rootContainer.children.forEach(control => {
    //       control.isVisible = getType
    //     })
    //   }
    // },

    selectMesh(meshId) {
      for (let key in this.downloadModelingMeshList) {
        const item = this.downloadModelingMeshList[key]

        for (let i = 0; i < item.mesh.meshes.length; i++) {
          if (item.mesh.meshes[i].id === meshId) {
            this.addoutline(item.mesh.meshes[i])
          }
        }
      }
    },
    addoutline(mesh) {
      if (mesh.isPickable) {
        this.selectionLayer.removeAllMeshes()
        this.selectionLayer.isEnabled = true
        this.selectionLayer.addMesh(mesh, new BABYLON.Color3(0, 0.64, 1))
        mesh.getChildMeshes().forEach(child => {
          this.selectionLayer.addMesh(child, new BABYLON.Color3(0, 0.64, 1))
        })
        this.selectionLayer.blurVerticalSize = 0.7
        this.selectionLayer.blurHorizontalSize = 0.7
        this.selectionLayer.innerGlow = false
      }
    },
    deselectMesh() {
      this.selectionLayer.removeAllMeshes()
    },
    createGridMaterial(scene, size) {
      this.ground = BABYLON.MeshBuilder.CreateGround('ground', { width: size, height: size }, scene)
      const gridMaterial = new GridMaterial('gridMaterial', scene)
      gridMaterial.majorUnitFrequency = 10
      gridMaterial.minorUnitVisibility = 1
      gridMaterial.gridRatio = 1
      gridMaterial.backFaceCulling = false
      gridMaterial.mainColor = new BABYLON.Color3(1, 1, 1)
      gridMaterial.lineColor = new BABYLON.Color3(0.5, 0.5, 0.5)
      gridMaterial.opacity = 0.5

      this.ground.material = gridMaterial
      this.ground.isVisible = false
      this.ground.isPickable = false
      return this.ground
    },
    createGizmo() {
      this.gizmoManager.positionGizmoEnabled = true
      this.gizmoManager.rotationGizmoEnabled = true
      this.gizmoManager.scaleGizmoEnabled = true

      this.gizmoManager.gizmos.positionGizmo.updateScale = false
      this.gizmoManager.gizmos.scaleGizmo.updateScale = false
      this.gizmoManager.gizmos.rotationGizmo.updateScale = false
      this.gizmoManager.gizmos.scaleGizmo.uniformScaleGizmo.updateScale = false // scale중심점

      this.allgizmofalse()
      this.gizmoManager.gizmos.scaleGizmo.sensitivity = 0.1

      //로컬 글로벌
      this.gizmoManager.gizmos.positionGizmo.updateGizmoRotationToMatchAttachedMesh = true
      this.gizmoManager.gizmos.scaleGizmo.updateGizmoRotationToMatchAttachedMesh = false
      this.gizmoManager.gizmos.rotationGizmo.updateGizmoRotationToMatchAttachedMesh = false

      this.gizmoManager.gizmos.scaleGizmo.scaleRatio = this.zoomLevel * 2
      this.gizmoManager.gizmos.rotationGizmo.scaleRatio = this.zoomLevel * 2
      this.gizmoManager.gizmos.positionGizmo.scaleRatio = this.zoomLevel * 2
    },
    allgizmofalse() {
      this.gizmoManager.positionGizmoEnabled = false
      this.gizmoManager.rotationGizmoEnabled = false
      this.gizmoManager.scaleGizmoEnabled = false
      this.gizmoManager.boundingBoxGizmoEnabled = false
    },
    gizmoOn(getType) {
      this.allgizmofalse()
      switch (getType) {
        case 'position':
          this.gizmoManager.positionGizmoEnabled = true
          this.gizmoManager.gizmos.positionGizmo.updateGizmoRotationToMatchAttachedMesh = true
          break
        case 'rotation':
          this.gizmoManager.rotationGizmoEnabled = true
          this.gizmoManager.gizmos.rotationGizmo.updateGizmoRotationToMatchAttachedMesh = true
          break
        case 'scale':
          this.gizmoManager.scaleGizmoEnabled = true
          this.gizmoManager.gizmos.scaleGizmo.updateGizmoRotationToMatchAttachedMesh = true
          break
      }
    },
    setEyes(isOn, child) {
      let findItem
      for (const key in this.downloadModelingMeshList) {
        if (this.downloadModelingMeshList[key].createId === child.createId) {
          findItem = this.downloadModelingMeshList[key]
        }
      }
      const meshes = findItem.mesh.meshes
      for (let i = 0; i < meshes.length; i++) {
        if (child.isRoot) {
          meshes[i].isVisible = isOn
          meshes[i].isPickable = isOn
          this.deselectMesh()
        } else if (meshes[i].id === child.id) {
          meshes[i].isVisible = isOn
          meshes[i].isPickable = isOn
          meshes[i]._children?.forEach(item => {
            item.isVisible = isOn
            item.isPickable = isOn
          })
          this.deselectMesh()
        }
      }
    },
    setLock(isOn, child) {
      let findItem
      for (const key in this.downloadModelingMeshList) {
        if (this.downloadModelingMeshList[key].meshId === child.meshId) {
          findItem = this.downloadModelingMeshList[key]
        }
      }
      const meshes = findItem.mesh.meshes
      for (let i = 0; i < meshes.length; i++) {
        if (child.isRoot) {
          meshes[i].isPickable = isOn
          this.deselectMesh()
        } else if (meshes[i].id === child.id) {
          meshes[i].isPickable = isOn
          this.deselectMesh()
        }
      }
    },
  },
}
</script>
