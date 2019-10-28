<template>
  <div class="SelectionGridContainer">
    {{q_selected}}..{{cell_selected}}
    <br/>
    {{qMoveFrom}}..{{qMoveTo}}
    <br/>
    <div v-for="(link,index) in map_links" :key="'link_'+index" >
      {{link.from}} - {{link.to}}..{{link.x1}},{{link.y1}},{{link.x2}},{{link.y2}}
    </div>
<!-- questions container -->
    <svg :width="QchartWidth" :height="QchartHeight" id="QuestionsContainer">
      <g
        v-for="block in map_array"
        :key="block.id"
        class="rect_question"
        @click="BlockClick(block.poz)"
        v-bind:class="{ qSelected: q_selected==block.poz,qUsed: block.used }"
      >
          <title v-if="!block.used">{{block.hover_text}}</title>
          <rect
          :x="block.pozx*q_width"
          :y="block.pozy*q_height"
          :width="q_width"
          :height="q_height"
        />
          <text
          :x="block.pozx*q_width+q_width/2"
          :y="block.pozy*q_height+q_height/2"
          alignment-baseline="middle"
          text-anchor="middle"
          class="text_question"
        >{{block.pozx}},{{block.pozy}},{{block.text}}</text>
        <!-- >{{block.poz}},{{block.move}},{{block.text}}</text> -->
        <!-- >{{block.text}}</text> -->
      </g>
    </svg>  

<!-- answers container -->
    <svg :width="AchartWidth" :height="AchartHeight" id="AnswersContainer">
      <g
        v-for="acell in map_answers"
        :key="acell.id"
        @click="CellClick(acell.poz)"
        @contextmenu="ContextMenuCell($event,acell.poz)"
        v-bind:class="{ cellSelected: acell.poz==qMoveFrom||acell.poz==qMoveTo}"
        class="rect_answers">

        <rect :x="acell.pozx*cell_width" :y="acell.pozy*cell_height" :width="cell_width" :height="cell_height" />
        <!-- <text :x="acell.pozx*cell_width+cell_width/2" :y="acell.pozy*cell_height+cell_height/2" alignment-baseline="middle" text-anchor="middle">{{acell.poz}}</text> -->
        <title v-for="(temp,index) in map_array.filter(item=>{return item.move==acell.poz})"
        :key="temp.id+index" >
         {{temp.hover_text}}
        </title>

        <text v-for="temp in map_array.filter(item=>{return item.move==acell.poz})"  
          :key="temp.id" 
          :x="acell.pozx*cell_width+cell_width/2"
          :y="acell.pozy*cell_height+cell_height/2"
          alignment-baseline="middle"
          text-anchor="middle"
        >{{temp.text}}</text>

      </g>
      <!-- links -->
    <g>
        <polyline   v-for="(link,index) in map_links" :key="'link_'+index"
          :points="GetPoints(link.x1,link.y1,link.x2,link.y2)"
          class="poly_line"
           @contextmenu="ContextMenuPloly($event,link.from,link.to)"
        />
   </g>
    </svg>

<!-- conditions container -->
    <div v-bind:style="{ width: CchartWidth+'px', height: CchartHeight + 'px' }" id="ConditionsContainer">

       <select class="DropDownCondition" id="DropDownCondition1" v-model="CD1">
        <option v-for="(question,questionIndex) in questions" :key="questionIndex" :value="question">{{ question }}</option>
      </select>
      <select class="DropDownCondition" id="DropDownCondition1" v-model="CD2">
        <option v-for="(operator,operatorIndex) in operators" :key="operatorIndex" :value="operator">{{ operator }}</option>
      </select>
      <select class="DropDownCondition" id="DropDownCondition1" v-model="CD3">
        <option v-for="(answer,answerIndex) in answers" :key="answerIndex" :value="answer">{{ answer }}</option>
      </select>


      <div class="CondButton" @click="CondClick(1)" v-if="!CB_visible">
          AND
      </div>
      <div class="CondButton"  @click="CondClick(2)" v-if="!CB_visible">
          OR
      </div>
      <div class="CondButton"  @click="CondClick(3)" v-if="!CB_visible">
          DONE
      </div>
      <div class="CondButton"  @click="CondClick(4)" v-if="CB_visible">
          GENERATE
      </div>
      <div class="CondButton"  @click="CondClick(5)" v-if="CB_visible">
          DELETE
      </div>
    </div>

    <!-- temporary conditions display container -->
    <div id="TemporaryConditions"  v-for="cond in tempCond" :key="cond.CD1+cond.CD2+cond.CD3+cond.id" >
      {{cond.CD1}} {{cond.CD2}} {{cond.CD3}}
      <div>
        {{cond.Next}}
      </div>   
    </div>


    <!-- conditions display container -->
        <div v-for="(condBlock,index1) in map_conditions" :key="index1" class="ConditionDiv">
        C{{index1}}:
              <div v-for="(cond,index2) in condBlock" :key="index2" >
                {{cond.CD1}} {{cond.CD2}} {{cond.CD3}} {{cond.Next}}
              </div>
      </div>
</div>
</template>

<script>
import SVG from "svg.js";

export default {
  name: "SelectionGrid2",
  data() {
    return {
      QchartHeight: 0,
      QchartWidth: 0,
      AchartHeight: 0,
      AchartWidth: 0,
      CchartHeight: 50,
      CchartWidth: 500,
      questions: ["START", "Q0", "Q1", "Q2", "Q3", "Q4", "Q5", "Q99", "STOP"],
      q_width: 75, //question width
      q_height: 50, //question height
      q_cols: 8, // number of questions per row
      q_rows: -1, //determined based on cols and count
      map_array:[],
      map_answers: [],
      map_conditions:[],
      operators:["=","<>","*"],
      answers:["ans1","ans2","ans3"],
      cell_width: 50, //question width
      cell_height: 50, //question height
      a_cols: 15, //answers_cols
      a_rows: 10, //answers_rows
      q_selected: -1,
      cell_selected: -1,
      nextqx:0,
      nextqy:0,
      CD1: "",
      CD2: "",
      CD3: "",
      tempCond:[],
      CB_visible: false,
      qMoveFrom:-1,
      qMoveTo: -1,
      map_links: [],
      qqq:0
    };
  },
  created() {
    this.Generate();
    var vueObj = this;

    $(window).resize(function() {
      console.log("resize");
    });
    // tbd
  },
  watch: {},
  methods: {
    Generate() {
      //add initial questions
      for (var i = 0; i < this.questions.length; i++) {
          this.AddBlock(this.questions[i],"");
      }

      // rezise answer container
      this.AchartWidth = this.a_cols * this.cell_width;
      this.AchartHeight = this.a_rows * this.cell_height;

      // generate answers
      for (var i = 0; i < this.a_cols * this.a_rows; i++) {
        this.map_answers.push({
          id: "cell_" + i,
          pozx: 0,
          pozy: 0,
          poz: 0,
          blocked: false 
        });
      }
      for (var i = 0; i < this.a_rows; i++) {
        for (var j = 0; j < this.a_cols; j++) {
          var item = i * this.a_cols + j;
          this.map_answers[item].pozx = j;
          this.map_answers[item].pozy = i;
          this.map_answers[item].poz = item;
        }
      } 
    },
    AddBlock(qtext,hoverText){
      //add question or condition in map_array
        var count=this.map_array.length;
        this.map_array.push({
          text: qtext,
          id: "block_" + count,
          pozx: this.nextqx,
          pozy: this.nextqy,
          used: false,
          poz: count,
          hover_text:hoverText,
          move: -1
        });
        if (this.nextqx<this.q_cols-1){
          this.nextqx++;
        }else{
        this.nextqx=0;
        this.nextqy++;
        }
        
      //get number of rows
      this.q_rows = Math.ceil(this.map_array.length / this.q_cols);
      // rezise question container
      this.QchartWidth = this.q_cols * this.q_width;
      this.QchartHeight = this.q_rows * this.q_height;
    },
    BlockClick(qpoz){
      // console.log('BlockClick for ',qpoz);
      if (!this.map_array[qpoz].used){
        this.q_selected=qpoz;
      }else{
        // console.log('question used');

      }
    },
    CellClick(poz){
        // console.log('selected cell');
        // 
        if (this.q_selected!=-1){
        //check if destination was previously selected
        
        if (!this.CheckEmptyCell(poz)) {       
          // console.log('cell already assigned');
          this.q_selected=-1;
          return;
        };
        // console.log('click on ', poz);
        this.cell_selected=poz;
        this.AssignCell();
      }else{
        // console.log('question not selected');
        this.CheckAndMove(poz);
      }
    },
    CheckAndMove(poz){
      if (this.qMoveFrom==-1 && !this.CheckEmptyCell(poz)) {
        this.qMoveFrom=poz;
        return;
        }
      if (this.qMoveFrom!=-1 && this.CheckEmptyCell(poz)) {
        this.qMoveTo=poz;
        var vueObj=this; 
        //move arrows
        do{
          var gasit=false;
          this.map_links.forEach(function(link,index) {
          if (link.from==vueObj.qMoveFrom){
            vueObj.UpdateLinks(vueObj.qMoveTo, link.to,index);
            gasit=true;
          } 
          if (link.to==vueObj.qMoveFrom) {
           vueObj.UpdateLinks(link.from, vueObj.qMoveTo,index);
           gasit=true;
          } 
         });   
        }while(gasit);
      
        //move question
        this.map_array.forEach(function(obj,index) {
          if (obj.move==vueObj.qMoveFrom) {
            vueObj.map_array[index].move=vueObj.qMoveTo;  
            vueObj.qMoveFrom=-1;
            vueObj.qMoveTo=-1;
          }  
        });
     
        return;
      }
      // console.log('create link in CheckAndMove');
      this.CreateLink(this.qMoveFrom, poz);
      this.qMoveFrom=-1;

    },
    UpdateLinks(newstart,newend,index){
        this.map_links.splice(index,1);
        this.CreateLink(newstart,newend);
    },
    CreateLink(start,end){
        // console.log('start, end: ',start, end);
        // console.log(this.a_cols,this.a_rows);
         this.map_links.push({
          from:start,
          to:end,
          x1: this.UpdatePoints(start,end,"startx"),
          y1: this.UpdatePoints(start,end,"starty"),
          x2: this.UpdatePoints(start,end,"endx"),
          y2: this.UpdatePoints(start,end,"endy")
        });
    },
    UpdatePoints(start,end,type){

      var startx=start;
      var starty=0;
      var endx=end;
      var endy=0;
      while (startx>this.a_cols-1) {
        startx=startx-this.a_cols;
        starty++;
      }
      while (endx>this.a_cols-1) {
        endx=endx-this.a_cols;
        endy++;
      }
    switch(type){
      case "startx": return startx; 
      case "starty": return starty;
      case "endx": return endx;  
      case "endy": return endy;
    }
    },
    CheckEmptyCell(poz){
        var isEmpty=true;
        this.map_array.forEach(function(obj,index) {
          if (obj.move==poz) {
            isEmpty=false;   
          }  
        });
        return isEmpty;
    },
    AssignCell(){
      this.map_array[this.q_selected].used=true;
      this.map_array[this.q_selected].move=this.cell_selected;
      this.q_selected=-1;
      this.cell_selected=-1;
    },
    CondClick(type){
      switch (type) {
          case 1:  //case AND
            this.tempCond.push({CD1:this.CD1, CD2:this.CD2, CD3:this.CD3, Next:"AND", id:this.tempCond.length-1});
            break;
          case 2:  //case OR
            this.tempCond.push({CD1:this.CD1, CD2:this.CD2, CD3:this.CD3, Next:"OR", id:this.tempCond.length-1});
            break; 
          case 3: //case DONE
            this.tempCond.push({CD1:this.CD1, CD2:this.CD2, CD3:this.CD3, Next:"", id:this.tempCond.length-1});
            this.CB_visible=true;
            break;   
          case 4: //case GENERATE
            this.map_conditions.push(this.tempCond);
            this.AddBlock("C"+this.map_conditions.length,this.MakeMePretty(this.map_conditions[this.map_conditions.length-1]));
          case 5: //case GENERATE and DELETE
            this.tempCond=[];
            this.CD1="";
            this.CD2="";
            this.CD3="";
            this.CB_visible=false;
            break;                                   
          default:
            console.log('error in type cond_click');
      }  
    },
    MakeMePretty(cond){
      var cond_pretty="";
      // console.log(cond.length);
      cond.forEach(function(condition,index) {
        cond_pretty=cond_pretty+condition.CD1+" "+condition.CD2+" "+condition.CD3+ " "+condition.Next +" "
      });
      // for (var i=0; i<cond.length; i++){
      //   cond_pretty=cond_pretty+cond.CD1+" "+cond.CD2+" "+cond.CD3+ " "+cond.Next+";"  
      // }
      return cond_pretty;
    },
    ContextMenuCell(e,cell){
      // console.log('context for ', cell);

      //delete arrows
      var vueObj=this;
      do{
        var gasit=false;
        this.map_links.forEach(function(link,index) {
        if (link.from==cell || link.to==cell) {
          vueObj.map_links.splice(index,1);
          gasit=true;
        } 
        });   
      }while(gasit);

      //delete cell
      this.map_array.forEach(function(obj,index) {
        if (obj.move==cell) {
          vueObj.map_array[index].move=-1; 
           vueObj.map_array[index].used=false;
        }  
      });
    
    
      e.preventDefault();
    },
    ContextMenuPloly(e,from,to){
      var vueObj=this;
      this.map_links.forEach(function(link,index) {
        if (link.from==from && link.to==to) {
          vueObj.map_links.splice(index,1);
        } 
        }); 
      e.preventDefault();
    },

    GetPoints(x1,y1,x2,y2){

      // console.log('Points:', x1,y1,x2,y2);
      var startType=0;
      var endType=0;
      // types
      //    2
      // 1     3
      //    4
      var px1=0;
      var py1=0;
      var px2=0;
      var py2=0;
      var px3=0;
      var py3=0;
      var px4=0;
      var py4=0;
      // console.log('----------------------');
      if (Math.abs(x1-x2)>1){
          if (x1<x2){
            startType=3;
            endType=1;
          }
          if (x1>x2){
            startType=1;
            endType=3;
          }
      }
      if (Math.abs(y1-y2)>1){
          if (y1<y2){
            startType=4;
            endType=2;
          }
          if (y1>y2){
            startType=2;
            endType=4;
          }
      }
      if (startType==0){
        if (y1<y2){
          startType=4;
        }
        if (y1>y2){
          startType=2;
        }
      }

      px1=this.returnPoints(startType,1, x1,y1,x2,y2);
      py1=this.returnPoints(startType,2, x1,y1,x2,y2);
      px2=this.returnPoints(startType,3, x1,y1,x2,y2);
      py2=this.returnPoints(startType,4, x1,y1,x2,y2);

      px3=this.returnPoints(endType,3, x2,y2,x1,y1);
      py3=this.returnPoints(endType,4, x2,y2,x1,y1);
      px4=this.returnPoints(endType,1, x2,y2,x1,y1);
      py4=this.returnPoints(endType,2, x2,y2,x1,y1);

      // console.log('startType:',startType);
      // console.log('endType:',endType);
      return(px1+","+py1+" "+px2+","+py2+ " "+px3+","+py3+" "+px4+","+py4);
      // return("0,40 40,40 40,80 80,80 80,120 120,120 120,160");
    },
    returnPoints(type,point, x1,y1,x2,y2){
 
       switch (type) {
          case 1:
            if (point==1) return x1*this.cell_width;
            if (point==2) return (y1+1)*this.cell_height-this.cell_height/2;
            if (point==3) return x1*this.cell_width-this.cell_width/2;
            if (point==4) return (y1+1)*this.cell_height-this.cell_height/2;
            break;
          case 2:
            if (point==1) return (x1+1)*this.cell_width-this.cell_width/2;
            if (point==2) return y1*this.cell_height;   
            if (point==3) return (x1+1)*this.cell_width-this.cell_width/2;
            if (point==4) return y1*this.cell_height-this.cell_height/2; 
            break;
          case 3:
            if (point==1) return (x1+1)*this.cell_width;
            if (point==2) return y1*this.cell_height+this.cell_height/2;   
            if (point==3) return (x1+1)*this.cell_width+this.cell_width/2;
            if (point==4) return y1*this.cell_height+this.cell_height/2;           
            break;
          case 4:
            if (point==1) return (x1+1)*this.cell_width-this.cell_width/2;
            if (point==2) return (y1+1)*this.cell_height;   
            if (point==3) return (x1+1)*this.cell_width-this.cell_width/2;
            if (point==4) return (y1+1)*this.cell_height+this.cell_height/2;             
            break;  
      }
    }
  }
};
</script>
<style>
#QuestionsContainer,
#AnswersContainer {
  border: 1px solid;
  display: flex;
}
#ConditionsContainer {
  border: 1px solid darkslategray;
}
.rect_question {
  fill: floralwhite;
  stroke-width: 1;
  stroke: #b3b3b3;
  cursor: pointer;
}

.text_question {
  fill: black;
  /* stroke: black ; */
  cursor: pointer;
  font-weight: bold;
  stroke-width: 0;
  font-size: small;
}
.rect_answers {
  fill: #e9f5f7;
  stroke-width: 1;
  stroke: #b3b3b3;
  cursor: pointer;
}
.CellBlocked {
  fill:#d0dcde;
  cursor: default;
}
.qSelected {
  fill: darkcyan;
}
.qUsed {
  fill: #b3b3b3;
  stroke-width: 1;
  stroke: #b3b3b3;
  cursor: no-drop;
}
.cellSelected {
  fill: darkcyan;
}
.arrow_triangle {
  stroke-width: 1;
  stroke: black;
}
.arrow_body {
  stroke-width: 3;
  stroke: black;
}

.arrow:hover * {
  stroke-width: 5;
  stroke: darkred !important;
  fill: darkred;
}
.CondButton{
  background-color: antiquewhite;
  border:1px solid;
  min-width: 100px;
  display: inline-flex;
  cursor: pointer;
}
.ConditionDiv{
  border:1px solid black;
  margin-top: 5px;
}
.poly_line{
    stroke-width: 3;
  stroke: black;
  fill:none
}
.poly_line:hover {
  stroke-width: 5;
  stroke: darkred !important;
}
</style>
