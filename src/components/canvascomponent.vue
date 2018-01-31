<template>
  <div>
    <h1>Mode: {{mode}}</h1>
    <button @click="selectBrush">Brush</button>
    <button @click="selectRect">Rectangle</button>
    <button @click="removeElement">Remove</button>
    <button @click="reset">Reset</button>
    <input v-model='brushWidth' />
    <input type="color" v-model='brushColor'>
  </div>
</template>

<script>
import 'fabric'
export default {

  name: 'canvascomponent',
  data() {
    return({
      mode: "Select",
      canvasObj: null,
      canvas: null,
      isDrawingMode: false,
      brushColor: '#000000',
      brushWidth: 5
    })
  },
  created() {
    let canvas = new Canvas('wtd_background_image');
    let canvasObj = new fabric.Canvas('wtd_background_image', { backgroundColor : "#afafaf", isDrawingMode: false });
    this.canvasObj = canvasObj
    function Canvas(id) {
      this.canvas = document.createElement('canvas');
      this.canvas.id = id;
      this.canvas.width = 500;
      this.canvas.height = 500;
      document.body.appendChild(this.canvas);
      return this.canvas;
    }
  },
  methods: {
    selectBrush() {
      this.reset()
      this.isDrawingMode = !this.isDrawingMode
      this.canvasObj.isDrawingMode = this.isDrawingMode
      this.mode = this.isDrawingMode ? 'Brush' : 'Select'
      if (this.isDrawingMode) {
        this.canvasObj.freeDrawingBrush.color = this.brushColor
        this.canvasObj.freeDrawingBrush.width = this.brushWidth
      }
    },
    removeElement() {
      let group = this.canvasObj.getActiveGroup()
      if (group) {
        group.forEachObject((o) => {
          this.canvasObj.remove(o)
        })
        this.canvasObj.discardActiveGroup().renderAll()
      }
      this.canvasObj.remove(this.canvasObj.getActiveObject())
    },
    selectRect() {
      this.isDrawingMode = false
      this.canvasObj.isDrawingMode = this.isDrawingMode

      let initialCoord = null
      this.canvasObj.on('mouse:down', (o) => {
        let clicked = true
        if (!initialCoord) initialCoord = {x: o.e.layerX, y: o.e.layerY}
          this.canvasObj.on('mouse:move', (o) => {
            if (clicked)
              console.log(o.e.layerX, o.e.layerY)
          })
        this.canvasObj.on('mouse:up', (o) => {
          clicked = false
          let rect = new fabric.Rect({left: initialCoord.x, top: initialCoord.y, width: o.e.layerX - initialCoord.x, height: o.e.layerY - initialCoord.y})
          this.canvasObj.add(rect)
          initialCoord = null
        })
      })
    },
    clearListeners() {
      this.canvasObj.__eventListeners["mouse:down"] = [];
      this.canvasObj.__eventListeners["mouse:move"] = [];
      this.canvasObj.__eventListeners["mouse:up"] = [];
    },
    reset() {
      try {
        this.clearListeners()
      } catch {

      }
    }
  }
}
</script>
