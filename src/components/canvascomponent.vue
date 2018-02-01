<template>
  <div>
    <div>
      <h1>Mode: {{mode}}</h1>
      <button @click="selectSelect">Select</button>
      <button @click="selectBrush">Brush</button>
      <button @click="selectRect">Rectangle</button>
      <button @click="selectCircle">Circle</button>
      <button @click="selectText">Text</button>
      <button @click="selectLine">Line</button>
      <button @click="removeElement" v-on:keyup.delete="removeElement">Remove</button>
      <button v-if='lastActions.length > 0' @click='undoLastRemove'>Unde Delete</button>
      <input v-model='brushWidth' />
      <input ref='colorPicker' type="color" :value='color' />
    </div>
    <div v-if='mode === "Text"'>
      Font Size:
      <input type="text" v-model='text.fontSize'/>
      Allign:
      <button name="left" v-bind:class="{ buttonActive: text.textAlign === 'left' }" @click="setAllignment">Left</button>
      <button name="center" v-bind:class="{ buttonActive: text.textAlign === 'center' }" @click="setAllignment">Center</button>
      <button name="right" v-bind:class="{ buttonActive: text.textAlign === 'right' }" @click="setAllignment">Right</button>
    </div>
    <div v-if='mode === "Line"'>
      Width:
      <input type="text" v-model='line.strokeWidth'/>
    </div>    
    <div v-if='mode === "Circle"'>
      Stroke Width:
      <input type="text" v-model='circle.strokeWidth'/>
    </div>
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
      brushWidth: 5,
      text: {
        fontSize: 10,
        textAlign: 'center',
        fontFamily: 'Arial'
      },
      line: {
        strokeWidth: 1
      },
      circle: {
        strokeWidth: 1
      },
      lastActions: []
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
    // DELETE elements when DLEETE os BACKSPACE is pressed
    window.addEventListener('keyup', (ev) => {
      if (ev.keyCode === 8 || ev.keyCode === 46) {
        this.removeElement()
      }
    });
  },
  methods: {
    selectBrush() {
      this.disableSelection()
      this.mode = 'Brush'
      this.isDrawingMode = true
      this.canvasObj.isDrawingMode = this.isDrawingMode
      this.canvasObj.freeDrawingBrush.color = this.color
      this.canvasObj.freeDrawingBrush.width = this.brushWidth
    },
    removeElement() {
      if (!this.canvasObj.getActiveGroup())
        this.lastActions.push(this.canvasObj.getActiveObject()) 
        this.canvasObj.remove(this.canvasObj.getActiveObject())
    },
    selectRect() {
      this.disableSelection()
      this.isDrawingMode = false
      this.canvasObj.isDrawingMode = this.isDrawingMode
      this.mode = 'Rectangle'
    },
    selectSelect() {
      this.enableSelection()
      this.isDrawingMode = false
      this.canvasObj.isDrawingMode = this.isDrawingMode
      this.mode = 'Select'
    },
    drawShape(data) {      
      if (this.mode === 'Rectangle') {
        this.drawRectangle(data)
      } else if(this.mode === 'Text') {
        this.drawTextBox(data)
      } else if(this.mode === 'Line') {
        this.drawLine(data)
      } else if(this.mode === 'Circle') {
        this.drawCircle(data)
      }
    },
    drawRectangle(data) {
      data.strokeWidth = 0
      data.fill = this.color
      let rect = new fabric.Rect(data)
      this.canvasObj.add(rect) 
    },
    selectText() {
      this.disableSelection()
      this.isDrawingMode = false
      this.canvasObj.isDrawingMode = this.isDrawingMode
      this.mode = 'Text'
    },
    drawTextBox(data) {
      data.fontSize = this.text.fontSize
      data.textAlign = this.text.textAlign
      data.fill = this.color
      data.fontFamily = this.text.fontFamily
      var text = new fabric.Textbox('Text...', data)
      this.lastActions.push(text)
      this.canvasObj.add(text);
      this.selectSelect()
    },
    setAllignment(e) {
      this.text.textAlign = e.target.name
    },
    selectLine() {
      this.disableSelection()
      this.isDrawingMode = false
      this.canvasObj.isDrawingMode = this.isDrawingMode
      this.mode = 'Line'
    },
    drawLine(data) {
      let x1 = data.left
      let x2 = data.left + data.width
      let y1 = data.top
      let y2 = data.top + data.height
      let options = {
        stroke: this.color
      }
      if (data.height >= 0){
        options.top = data.top
      } else {
        options.bottom = data.top
      }
      if (data.width >= 0) {
        options.left = data.left
      } else {
        options.right = data.left
      }
      options.strokeWidth = parseInt(this.line.strokeWidth)
      let line = new fabric.Line([x1, y1, x2, y2], options)
      this.lastActions.push(line)
      this.canvasObj.add(line)
    },
    disableSelection() {
      this.canvasObj.forEachObject(function(o) {
        o.selectable = false;
      });
    },
    enableSelection() {
      this.canvasObj.forEachObject(function(o) {
        o.selectable = true;
      });
    },
    selectCircle() {
      this.disableSelection()
      this.isDrawingMode = false
      this.canvasObj.isDrawingMode = this.isDrawingMode
      this.mode = 'Circle'
    },
    drawCircle(data) {
      const radius = data.width / 2
      let options = {
        radius: radius,
        top: data.top,
        left: data.left,
        fill: this.color,
        strokeWidth: parseInt(this.circle.strokeWidth)
      }
      let circle = new fabric.Circle(options)
      this.lastActions.push(circle)
      this.canvasObj.add(circle)
    },
    undoLastRemove() {
      const item = this.lastActions.slice(-1).pop()
      this.lastActions.pop()
      this.canvasObj.add(item)
    }
  }
}
</script>

<style type="text/css" scoped>
.buttonActive {
  background-color: white;
}
</style>