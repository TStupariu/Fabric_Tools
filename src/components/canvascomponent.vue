<template>
  <div>
    <h1>Mode: {{mode}}</h1>
    <button @click="selectSelect">Select</button>
    <button @click="selectBrush">Brush</button>
    <button @click="selectRect">Rectangle</button>
    <button @click="removeElement">Remove</button>
    <input v-model='brushWidth' />
    <input ref='colorPicker' type="color" :value='color' />
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
      color: '#000000',
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
  mounted() {
    // SHAPE DRAWING
    let initialCoord = null
    this.canvasObj.on('mouse:down', (o) => {
      if (!initialCoord) {
        initialCoord = {x: o.e.layerX, y: o.e.layerY}
      }
      this.canvasObj.on('mouse:up', (o) => {
        if (initialCoord)
          this.drawShape({left: initialCoord.x, top: initialCoord.y, width: o.e.layerX - initialCoord.x, height: o.e.layerY - initialCoord.y })
        initialCoord = null
      })
    })
    // Color Picker
    this.$refs.colorPicker.addEventListener('input', (event) => {
      this.color = event.target.value
      this.canvasObj.freeDrawingBrush.color = this.color
    })
  },
  methods: {
    selectBrush() {
      this.mode = 'Brush'
      this.isDrawingMode = true
      this.canvasObj.isDrawingMode = this.isDrawingMode
      this.canvasObj.freeDrawingBrush.color = this.color
      this.canvasObj.freeDrawingBrush.width = this.brushWidth
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
      this.mode = 'Rectangle'
    },
    selectSelect() {
      this.isDrawingMode = false
      this.canvasObj.isDrawingMode = this.isDrawingMode
      this.mode = 'Select'
    },
    drawShape(data) {
      if (this.mode === 'Rectangle') {
        this.drawRectangle(data)
      }
    },
    drawRectangle(data) {
      data.strokeWidth = 0
      data.fill = this.color
      let rect = new fabric.Rect(data)
      console.log(rect)
      this.canvasObj.add(rect) 
    }
  }
}
</script>
