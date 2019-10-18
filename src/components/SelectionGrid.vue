<template>
  <div class="SelectionGridContainer">
 <svg :width="chartWidth" :height="chartHeight" id="SVGtag">

      <!-- create rect questions -->
      <g v-for="(qcell,qIndex) in map_questions" :key="qcell.id"  v-bind:class="{ qSelected: qIndex==q_selected, qUsed: qcell.used }" @click="decide_select_q(qIndex)">
          <rect         
            :x="qcell.x" 
            :y="qcell.y"
            :width="line_wh" 
            :height="line_wh"  
            :text="qcell.text"
            class="myQuestion"         
          >
          </rect>
           <text            
            :x="qcell.x+line_wh/2" 
            :y="qcell.y+line_wh/2"
             alignment-baseline="middle" text-anchor="middle"
             class="myQText">{{qcell.text}}</text>
          </g>
      <!-- create rect map-->
      <g v-for="(cell,index) in map_array" :key="index" v-bind:class="{ cellSelected: cell.index==cell_selected }" @click="decide_select_cell(index)" @contextmenu="ContextMenuCell($event,cell)">
          <rect           
            :x="cell.x" 
            :y="line_wh+cell.y"
            :width="line_wh" 
            :height="line_wh"
            :indx="cell.index"  
            class="myCell" 
          >
          </rect>

                    <polygon 
          :points="GeneratePoints(cell.x,line_wh+cell.y,0,cell.shape)" style="stroke-width:1"
          v-bind:class="{ poly1: cell.shape=='rectangle', poly2: cell.shape=='paralelogram', poly3: cell.shape=='rhombus' }"
          >
          </polygon>

               <text            
            :x="cell.x+line_wh/2" 
            :y="line_wh+cell.y+line_wh/2"
             alignment-baseline="middle" text-anchor="middle"
             class="myQText">{{cell.text}}</text>



      </g>
      <!-- create menu -->
      <g v-if="menu_visible" class="myContextMenu">
        <rect 
        :x="menux" 
        :y="menuy"
        :width="line_wh" 
        :height="line_wh*nr_shapes" >
          </rect>
     
          <polygon  v-for="(shape,index) in shapes_order" :key="index"
          :points="GeneratePoints(menux,menuy,index,shape)" style="stroke-width:1"
          @click="SelectShape(shape)"
          v-bind:class="{ poly1: shape=='rectangle', poly2: shape=='paralelogram', poly3: shape=='rhombus' }"
          >
          </polygon>
           
    
</g>
    </svg>
  </div>

</template>

<script>
import SVG from "svg.js";

export default {
  name: "SelectionGrid",
  data() {
    return {
   
      chartHeight: 300,
      chartWidth: 300,
      start_cols: 10,
      start_rows: 5,
      line_wh:75,
      map_array: [],
      map_questions:[],
      questions:["Q0","Q1","Q2","Q99"],
      nr_rows: 0,
      nr_cols: 0,
      max_rows:50,
      max_cols: 20,
      q_selected: -1,
      cell_selected: -1,
      menux:-100,
      menuy:-100,
      menu_visible: true,
      nr_shapes: 3,
      shapes_order: ["rectangle","paralelogram","rhombus"],
      shapes:{
        rectangle: {
        top1:10,
        top2:-1,
        left1:5,
        left2:-1,
      },
       paralelogram: {
        top1:10,
        top2:-1,
        left1:5,
        left2:15,
      },
      rhombus:  {
        top1:10,
        top2:-1,
        left1:5,
        left2:-1,
      },
      none:  {
        top1:0,
        top2:0,
        left1:0,
        left2:0,
      }
      }
    };
  },
  created() {

    this.Generate();
    var vueObj = this;
    $(window).resize(function() {
      console.log('resize');
    });

  },
  watch:{
    nr_rows(value){
      this.Resize_display();
    },
    nr_cols(value){
      this.Resize_display();
    },
    menu_visible(value){
      if (value==false) this.cell_selected=-1;
    }
  },
  methods: {
    SelectShape(shape){
      this.map_array[this.cell_selected].shape=shape;
      this.menu_visible=false;
    },
    GeneratePoints(x,y,index,shape){
      var points_string="";

      var top1=this.shapes[shape].top1;
      var top2=this.shapes[shape].top2;
      var left1=this.shapes[shape].left1;
      var left2=this.shapes[shape].left2;

      var points=[];


      switch(shape) {
        case "rectangle":
          points[0]=x+left1;
          points[1]=y+top1+index*this.line_wh;
          points[2]=x+left1;
          points[3]=(y+top1+index*this.line_wh+y-top1+(index+1)*this.line_wh)/2;
          points[4]=x+left1;
          points[5]=y-top1+(index+1)*this.line_wh;
          points[6]=x+this.line_wh/2;
          points[7]=y-top1+(index+1)*this.line_wh;
          points[8]=x+this.line_wh-left1;
          points[9]=y-top1+(index+1)*this.line_wh; 
          points[10]=x+this.line_wh-left1;
          points[11]=(y+top1+index*this.line_wh+y-top1+(index+1)*this.line_wh)/2;
          points[12]=x+this.line_wh-left1;
          points[13]=y+top1+index*this.line_wh;
          points[14]=x+this.line_wh/2;
          points[15]=y+top1+index*this.line_wh;
          break;
        case "paralelogram":

          points[0]=x+left2;
          points[1]=y+top1+index*this.line_wh;
          points[2]=x+(left1+left2)/2;
          points[3]=(y+top1+index*this.line_wh+y-top1+(index+1)*this.line_wh)/2;
          points[4]=x+left1;
          points[5]=y-top1+(index+1)*this.line_wh;
          points[6]=x+this.line_wh/2;
          points[7]=y-top1+(index+1)*this.line_wh;
          points[8]=x+this.line_wh-left2;
          points[9]=y-top1+(index+1)*this.line_wh; 
          points[10]=x+this.line_wh-(left1+left2)/2;
          points[11]=(y+top1+index*this.line_wh+y-top1+(index+1)*this.line_wh)/2;
          points[12]=x+this.line_wh-left1;
          points[13]=y+top1+index*this.line_wh;
          points[14]=x+this.line_wh/2;
          points[15]=y+top1+index*this.line_wh;
          break;
        case "rhombus":

          points[0]=x+this.line_wh/2;
          points[1]=y+top1+index*this.line_wh;
          points[2]=x+left1;
          points[3]=(y+top1+index*this.line_wh+y-top1+(index+1)*this.line_wh)/2;
          points[4]=x+this.line_wh/2;
          points[5]=y-top1+(index+1)*this.line_wh;
          points[6]=x+this.line_wh/2;
          points[7]=y-top1+(index+1)*this.line_wh;
          points[8]=x+this.line_wh/2;
          points[9]=y-top1+(index+1)*this.line_wh; 
          points[10]=x+this.line_wh-left1;
          points[11]=(y+top1+index*this.line_wh+y-top1+(index+1)*this.line_wh)/2;
          points[12]=x+this.line_wh/2;
          points[13]=y+top1+index*this.line_wh;
          points[14]=x+this.line_wh/2;
          points[15]=y+top1+index*this.line_wh;

          break;          
        default:
          points[0]=0;
          points[1]=0;
          points[2]=0;
          points[3]=0;
          points[4]=0;
          points[5]=0;
          points[6]=0;
          points[7]=0;
          points[8]=0;
          points[9]=0; 
          points[10]=0;
          points[11]=0;
          points[12]=0;
          points[13]=0;
          points[14]=0;
          points[15]=0;
      }
      points_string=points[0].toString()+","+points[1].toString()+" "+points[2].toString()+","+points[3].toString()+" "+points[4].toString()+","+points[5].toString()+" "+points[6].toString()+","+points[7].toString()+" "+points[8].toString()+","+points[9].toString()+" "+points[10].toString()+","+points[11].toString()+" "+points[12].toString()+","+points[13].toString()+" "+points[14].toString()+","+points[15].toString()+" ";
      return(points_string);
    },
    Generate() {
      // generate questions
       for (var i=0; i<this.questions.length; i++){
          this.map_questions.push({x:i*this.line_wh,y:0,text:this.questions[i],id:"question_"+i,used:false});
       };
      // generate map
      this.nr_rows=this.start_rows;
      this.nr_cols=this.start_cols;
      this.Resize_display();

      console.log('generate');
      for (var i=0; i<this.max_rows; i++){
        for (var j=0; j<this.max_cols; j++){
          this.map_array.push({x:i*this.line_wh,y:j*this.line_wh,text:"",index:i*this.max_cols+j,shape:"none"});
        }
      }
    },
    Resize_display(){
      this.chartHeight=(this.nr_rows+1)*this.line_wh;
      this.chartWidth=this.nr_cols*this.line_wh;
    },
    decide_select_q(qIndex){
      this.menu_visible=false;
      if (this.q_selected==qIndex || this.map_questions[qIndex].used){
        this.q_selected=-1;
      }else{
        this.q_selected=qIndex;
      }
      
    },
    decide_select_cell(index){
      this.menu_visible=false;
       if (this.q_selected!=-1){
         console.log('se poate');
         this.map_array[index].text=this.questions[this.q_selected];
         this.map_questions[this.q_selected].used=true;
         this.q_selected=-1;
       }else{
         console.log('try again, select question first');
       }
    },
      ContextMenuCell: function(e,cell) {
        //do stuff
        
        if (cell.text.length>0){
          this.menu_visible=true;
          this.cell_selected=cell.index;
        }else{
          this.menu_visible=false;
        }
        if (cell.x<this.chartWidth/2){
          this.menux=cell.x+this.line_wh;//left a row
        }else{
          this.menux=cell.x-this.line_wh;//right  a row
        }
        if (cell.y<this.chartHeight/2){
          this.menuy=cell.y+this.line_wh;//down a row for header
        }else{
          this.menuy=cell.y+this.line_wh*(2-this.nr_shapes);//up one row
        }

        
        
        e.preventDefault();
     }
  }
};
</script>
<style>

#SVGtag{
  border:1px solid ;
}
.myCell{
   fill: #c1e4e4; /* #49bfbc;*/
  stroke-width: 1;
  stroke: #b3b3b3;
  cursor: pointer;

}
.myQuestion{
   fill: floralwhite; /* #49bfbc;*/
  stroke-width: 1;
  stroke: #b3b3b3;
  cursor: pointer;
}
.myQText{
  fill: black;
  cursor: pointer;
}

.qSelected> .myQuestion{
  fill: #61bfbf;
  cursor: pointer;
}
.qUsed> .myQuestion{
  fill: gray;
}

.cellSelected> .myCell{
  fill:cadetblue;
}
.myContextMenu{
  fill:cornsilk;
  stroke: black;
  cursor: pointer;
}

.poly1{
    fill: cadetblue;
    stroke: black;
    stroke-width: 1px;
    stroke-linejoin: round;
}
.poly2{
    fill: cadetblue;
    stroke: black;
    stroke-width: 1px;
    stroke-linejoin: round;
}
.poly3{
    fill: cadetblue;
    stroke: black;
    stroke-width: 1px;
    stroke-linejoin: round;
}
</style>
