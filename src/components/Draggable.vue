<template>


<div class="container ui-widget-content"  >
    <div class="draggable selector" id="elem1">
    Elem1
  </div>
  <div class="draggable selector" id="elem2">
    Elem2
  </div>
    <div class="draggable selector" id="elem3">
    Elem3
  </div>
</div>


</template>

<script>
import SVG from "svg.js";
import '@svgdotjs/svg.draggable.js'

export default {
  name: "Draggable",
  data() {
    return {
   
      chartHeight: 300,
      chartWidth: 300
    };
  },
  created() {

    this.Generate();
    var vueObj = this;
    $(window).resize(function() {
      console.log('resize');
    });

  },
  mounted(){

  $( function() {
    $( ".draggable" ).draggable();
  } );

  $( ".selector" ).draggable({
  containment: "parent"
  });
   $( ".selector" ).draggable({
     start: function( event, ui ) {
      console.log("start",ui.helper[0].id);
      console.log($(this).attr('id'),ui.position.top,ui.position.left);
      console.log($(this).attr('id'),ui.offset.top,ui.offset.left);       
     },
    stop: function(event, ui ) {
      console.log("stop",ui.helper[0].id);
      console.log($(this).attr('id'),ui.position.top,ui.position.left);
      console.log($(this).attr('id'),ui.offset.top,ui.offset.left);
    }
  });
$( ".selector" ).draggable({
  grid: [ 100 , 100 ]
});

  

  },
  watch:{
  },
  methods: {
    dragstop(){
      console.log("AAA");
    },
    Generate() {

      var draw = SVG().addTo('#SVGtag').size(400, 400)
      var rect = draw.rect(100, 100)

      rect.draggable()

    }
  }
};
</script>
<style>

.container{
  width:500px;
  height:500px;
  border:1px solid red;
}
.draggable { width: 75px; height: 75px; margin-bottom: 25px;}
#elem1{
  background-color: cadetblue;
}
#elem2{
  background-color: darkcyan;
}
#elem3{
  background-color: darkolivegreen;
}
.draggable {
  cursor: move;
}
</style>
