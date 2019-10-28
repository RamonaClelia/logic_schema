<template>
  <div class="SelectionGridContainer">
    <!-- questions container -->
    <svg :width="QchartWidth" :height="QchartHeight" id="QuestionsContainer">
      <g
        v-for="qcell in map_questions"
        :key="qcell.id"
        class="rect_question"
        @click="QuestionClick(qcell.pozx,qcell.pozy,qcell.used,qcell.poz)"
      >
        <title v-if="!qcell.used">{{qcell.hover_text}}</title>
        <rect
          :x="qcell.pozx*q_width"
          :y="qcell.pozy*q_height"
          :width="q_width"
          :height="q_height"
          v-bind:class="{ qSelected: q_selected==qcell.poz,qUsed: qcell.used }"
        />
        <text
          :x="qcell.pozx*q_width+q_width/2"
          :y="qcell.pozy*q_height+q_height/2"
          alignment-baseline="middle"
          text-anchor="middle"
          v-bind:class="{ qUsed: qcell.used }"
          class="text_question"
        >{{qcell.text}}</text>
        <!-- >{{qcell.text}}..{{qcell.pozx}} {{qcell.pozy}} {{qcell.poz}}</text> -->
      </g>
    </svg>   
    <!-- answers container -->
    <svg :width="AchartWidth" :height="AchartHeight" id="AnswersContainer">
      <g
        v-for="acell in map_answers"
        :key="acell.id"
        class="rect_answers"
        @click="AnswerClick(acell.pozx,acell.pozy,acell.poz)"
        @contextmenu="ContextMenuCell($event,acell.poz)"
        v-bind:class="{ cellSelected: acell.poz==ans1||acell.poz==ans2 ,CellBlocked:acell.blocked}"
      >
<!-- <title v-if="acell.text">{{map_questions[0].text}} {{acell.text}}</title> -->
        
        <title v-for="qcell in map_questions.filter(item=>{return item.text==acell.text})" 
        :key="qcell.id" >
         {{qcell.hover_text}}
          </title>
         
        <rect :x="acell.pozx*cell_width" :y="acell.pozy*cell_height" :width="cell_width" :height="cell_height" />
        <text
          :x="acell.pozx*cell_width+cell_width/2"
          :y="acell.pozy*cell_height+cell_height/2"
          alignment-baseline="middle"
          text-anchor="middle"
          class="text_question"
        >{{acell.text}}</text>
      </g>
      <g
        v-for="(link,linkIndex) in links"
        :key="linkIndex"
        class="arrow"
        @contextmenu="ContextMenuArrow($event,linkIndex)"
      >
         <line
          v-if="link.display && link.typeStart==1"        
          :x1="link.x1-cell_width/2"  :y1="link.y1"   :x2="link.x1"   :y2="link.y1"
          stroke-width="1"    stroke="black"     class="arrow_body"
        /> 
         <line
          v-if="link.display && link.typeStart==2"        
          :x1="link.x1"  :y1="link.y1-cell_height/2"   :x2="link.x1"   :y2="link.y1"
          stroke-width="1"    stroke="black"     class="arrow_body"
        /> 
                 <line
          v-if="link.display && link.typeStart==3"        
          :x1="link.x1+cell_width/2"  :y1="link.y1"   :x2="link.x1"   :y2="link.y1"
          stroke-width="1"    stroke="black"     class="arrow_body"
        /> 
                 <line
          v-if="link.display && link.typeStart==4"        
          :x1="link.x1"  :y1="link.y1+cell_height/2"   :x2="link.x1"   :y2="link.y1"
          stroke-width="1"    stroke="black"     class="arrow_body"
        />   
        <line
          v-if="link.display"
          :x1="link.x1"
          :y1="link.y1"
          :x2="link.x2"
          :y2="link.y2"
          stroke-width="1"
          stroke="black"
          class="arrow_body"
        />
                 <line
          v-if="link.display && link.typeEnd==1"        
          :x1="link.x2-cell_width/2+triangle_height"  :y1="link.y2"   :x2="link.x2"   :y2="link.y2"
          stroke-width="1"    stroke="black"     class="arrow_body"
        /> 
         <line
          v-if="link.display && link.typeEnd==2"        
          :x1="link.x2"  :y1="link.y2-cell_height/2+triangle_height"   :x2="link.x2"   :y2="link.y2"
          stroke-width="1"    stroke="black"     class="arrow_body"
        /> 
                 <line
          v-if="link.display && link.typeEnd==3"        
          :x1="link.x2+cell_width/2-triangle_height"  :y1="link.y2"   :x2="link.x2"   :y2="link.y2"
          stroke-width="1"    stroke="black"     class="arrow_body"
        /> 
                 <line
          v-if="link.display && link.typeEnd==4"        
          :x1="link.x2"  :y1="link.y2+cell_height/2-triangle_height"   :x2="link.x2"   :y2="link.y2"
          stroke-width="1"    stroke="black"     class="arrow_body"
        />   
        <polygon
          v-if="link.display"
          :points="link.arrow.p1+','+link.arrow.p2+' '+link.arrow.p3+','+link.arrow.p4+' '+link.arrow.p5+','+link.arrow.p6"
          class="arrow_triangle"
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

    <!-- to be deleted -->


    <!-- {{ans1}},{{ans2}} -->
    <!-- <div
      v-for="(link,linkIndex) in links"
      :key="linkIndex"
    >{{link.index}}...{{link.start}},{{link.end}}...{{link.x1}},{{link.y1}},..,{{link.x2}},{{link.y2}}....{{link.display}}</div> -->
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
      map_questions: [],
      map_answers: [],
      map_conditions:[],
      operators:["=","<>","*"],
      answers:["ans1","ans2","ans3"],
      questions: ["START", "Q0", "Q1", "Q2", "Q3", "Q4", "Q5", "Q99", "STOP"],
      q_width: 50, //question width
      q_height: 50, //question height
      q_cols: 10, // number of questions per row
      q_rows: -1, //determined based on cols and count
      next_Qpozx:-1,
      next_Qpozy:-1,
      next_Qpoz:-1,
      cell_width: 50, //question width
      cell_height: 50, //question height
      a_cols: 15, //answers_cols
      a_rows: 10, //answers_rows
      q_selected: -1,
      links: [],
      ans1: -1,
      ans2: -1,
      triangle_base: 20,
      triangle_height: 10,
      last_ind: 0,
      CD1: "",
      CD2: "",
      CD3: "",
      tempCond:[],
      CB_visible: false
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
      // generate questions
      for (var i = 0; i < this.questions.length; i++) {
        this.map_questions.push({
          x: 0,
          y: 0,
          text: this.questions[i],
          id: "question_" + i,
          pozx: 0,
          pozy: 0,
          used: false,
          poz: 0,
          hover_text:""
        });
      }
      this.q_rows = Math.ceil(this.questions.length / this.q_cols);
      var item = 0;
      //  console.log(this.questions.length,this.q_cols,this.q_rows);
      for (var i = 0; i < this.q_rows; i++) {
        for (var j = 0; j < this.q_cols; j++) {
          if (item < this.questions.length) {
            this.map_questions[item].pozx = j;
            this.map_questions[item].pozy = i;
            this.map_questions[item].poz = item;
            // console.log(i,j,this.map_questions[item].text);
            item++;
          }
        }
      }
      this.next_Qpoz=item-1;
      this.GetNextPoz();
      
      // rezise question container
      this.QchartWidth = this.q_cols * this.q_width;
      this.QchartHeight = this.q_rows * this.q_height;

      // generate answers
      for (var i = 0; i < this.a_cols * this.a_rows; i++) {
        this.map_answers.push({
          x: 0,
          y: 0,
          text: "",
          id: "cell_" + i,
          pozx: 0,
          pozy: 0,
          poz: 0,
          blocked: false 
        });
      }
      //  console.log(this.map_answers.length,this.a_cols,this.a_rows);
      for (var i = 0; i < this.a_rows; i++) {
        for (var j = 0; j < this.a_cols; j++) {
          var item = i * this.a_cols + j;
          this.map_answers[item].pozx = j;
          this.map_answers[item].pozy = i;
          this.map_answers[item].poz = item;
        }
      }
      // rezise answer container
      this.AchartWidth = this.a_cols * this.cell_width;
      this.AchartHeight = this.a_rows * this.cell_height;
    },
    GetNextPoz(){
      
      console.log(this.map_questions[this.next_Qpoz].pozx,this.map_questions[this.next_Qpoz].pozy);
      if (this.map_questions[this.next_Qpoz].pozx==this.q_cols-1){
           this.next_Qpozx=0;
           this.next_Qpozy=this.map_questions[this.next_Qpoz].pozy+1;
      }else{
          this.next_Qpozx=this.map_questions[this.next_Qpoz].pozx+1;
          this.next_Qpozy=this.map_questions[this.next_Qpoz].pozy;
      }
      // rezise question container height if needed
      if (this.next_Qpozx==1){
          this.q_rows=this.q_rows+1;
          this.QchartHeight = this.q_rows * this.q_height;
      }
      
      console.log(' this.next_Qpozx:', this.next_Qpozx);   
      console.log(' this.next_Qpozy:', this.next_Qpozy); 

      this.next_Qpoz=this.next_Qpoz+1;

      


    },
    QuestionClick(pozx, pozy, used, poz) {
      if (this.q_selected==poz) {
        this.q_selected=-1;
      }else{
        if (!used) {
          this.q_selected = poz;
          this.ans1=-1;
          // console.log("question selected");
        } else {
          console.log("question already used");
        }
      }
    //generate Conditions container
  
    },
    AnswerClick(pozx, pozy, poz) {
     
      if (this.q_selected != -1) {
         if (!this.map_answers[poz].blocked){
        if (this.map_answers[poz].text == "") {
          this.map_answers[poz].text = this.map_questions[this.q_selected].text;
          this.map_questions[this.q_selected].used = true;
          this.q_selected = -1;
          this.MarkBlock();
        } else {
          console.log("already has text");
        }

}else{
        console.log('blocked Cell');
      }
      } else {
        console.log("no question selected");
        if (this.map_answers[poz].text != "") {
          if (this.ans1 == -1) {
            this.ans1 = poz;
          } else {
            if (this.ans2 == -1) {
              this.ans2 = poz;
              if (this.ans1 != this.ans2) {
                this.CheckAndCreateArrow(this.ans1,this.ans2);
              } else {
                console.log("don`t select the same cell");
              }
              this.ans1 = -1;
              this.ans2 = -1;
            } else {
              this.ans1 = poz;
              this.ans2 = -1;
            }
          }
        } else {
          console.log("no question selected and empty answer cell");
          // console.log(this.ans1);
          if (this.ans1 != -1) {
            // console.log("move from ", this.ans1, " to ", poz);
            this.SwapCells(this.ans1, poz);
          }
        }
      }
      
    },
    SwapCells(cell1, cell2) {
      if (!this.map_answers[cell2].blocked){
      // console.log('make it swap ', cell1, cell2);
      this.map_answers[cell2].text=this.map_answers[cell1].text
      this.map_answers[cell1].text="";
      this.MarkBlock();
      this.ans1=-1;
      this.CloneAllLinks(cell1,cell2);
      this.DeleteAllLinks(cell1);
      }else{
        console.log('destination cell blocked');
      }
    },
    CloneAllLinks(cell1,cell2){
      var vueObj=this;
      this.links.forEach(function(link,index) {
        if (link.start==cell1) {
          vueObj.CheckAndCreateArrow(cell2,link.end)
        }
        if (link.end==cell1) {
          vueObj.CheckAndCreateArrow(link.start,cell2)
        }  
      });

    },
    CheckAndCreateArrow(cell1,cell2){
          if (this.CheckArrow(cell1, cell2)) {
                this.last_ind++;
                this.links.push({
                  start: cell1,
                  end: cell2,
                  x1: 0,
                  y1: 0,
                  x2: 0,
                  y2: 0,
                  index: this.last_ind,
                  arrow: {},
                  typeStart:0,
                  typeEnd:0,
                  display: true
                });
                this.GenerateArrowPath(
                  cell1,
                  cell2,
                  this.links.length - 1
                );
              } else {
                console.log("failed to CheckArrow");
              }
    },
    GenerateArrowPath(ans1, ans2, index) {
      var x1 = this.map_answers[ans1].pozx;
      var y1 = this.map_answers[ans1].pozy;
      var x2 = this.map_answers[ans2].pozx;
      var y2 = this.map_answers[ans2].pozy;
      // console.log(x1,y1,x2,y2);
      var LR = "undef";
      var UD = "undef";
      if (x1 < x2) LR = "LR";
      if (x1 > x2) LR = "RL";
      if (x1 == x2) LR = "same";

      if (y1 < y2) UD = "UD";
      if (y1 > y2) UD = "DU";
      if (y1 == y2) UD = "same";

      //default left-up corner
      this.links[index].x1 = this.map_answers[ans1].pozx * this.cell_width;
      this.links[index].y1 = this.map_answers[ans1].pozy * this.cell_height;
      this.links[index].x2 = this.map_answers[ans2].pozx * this.cell_width;
      this.links[index].y2 = this.map_answers[ans2].pozy * this.cell_height;

      this.links[index].x1 = -1;
      this.links[index].y1 = -1;
      this.links[index].x2 = -1;
      this.links[index].y2 = -1;

      var halfWidth=this.cell_width/2;
      var halfHeight=this.cell_height/2;

      if (LR == "LR") {
        this.links[index].x1 = (this.map_answers[ans1].pozx + 1) * this.cell_width+halfWidth;
        this.links[index].y1 = this.map_answers[ans1].pozy * this.cell_height + this.cell_height / 2;

        this.links[index].x2 = this.map_answers[ans2].pozx * this.cell_width-halfWidth;
        this.links[index].y2 = this.map_answers[ans2].pozy * this.cell_height + this.cell_height / 2;

        this.CreateArrow(index, "left");
        this.links[index].typeStart=1;
        this.links[index].typeEnd=3;
      }

      if (LR == "RL") {
        this.links[index].x1 = this.map_answers[ans1].pozx * this.cell_width-halfWidth;
        this.links[index].y1 = this.map_answers[ans1].pozy * this.cell_height + this.cell_height / 2;

        this.links[index].x2 = (this.map_answers[ans2].pozx + 1) * this.cell_width+halfWidth;
        this.links[index].y2 = this.map_answers[ans2].pozy * this.cell_height + this.cell_height / 2;

        this.CreateArrow(index, "right");

        this.links[index].typeStart=3;
        this.links[index].typeEnd=1;
      }

      if (LR == "same") {
        if (UD == "UD") {
          this.links[index].x1 = this.map_answers[ans1].pozx * this.cell_width + this.cell_width / 2;
          this.links[index].y1 = (this.map_answers[ans1].pozy + 1) * this.cell_height+halfHeight;

          this.links[index].x2 = this.map_answers[ans2].pozx * this.cell_width + this.cell_width / 2;
          this.links[index].y2 = this.map_answers[ans2].pozy * this.cell_height-halfHeight;

        this.links[index].typeStart=2;
        this.links[index].typeEnd=4;

          this.CreateArrow(index, "up");
        }
        if (UD == "DU") {
          this.links[index].x1 = this.map_answers[ans1].pozx * this.cell_width + this.cell_width / 2;
          this.links[index].y1 = this.map_answers[ans1].pozy * this.cell_height-halfHeight;

          this.links[index].x2 = this.map_answers[ans2].pozx * this.cell_width + this.cell_width / 2;
          this.links[index].y2 = (this.map_answers[ans2].pozy + 1) * this.cell_height+halfHeight;

        this.links[index].typeStart=4;
        this.links[index].typeEnd=2;

          this.CreateArrow(index, "down");
        }
      }
      // correct LR and RL Z

      if (x1==x2+1 || x1==x2-1) {
              console.log('arrow Z correction');
 
              if (y1<y2){
                // console.log('case1');
                this.links[index].x1 = this.map_answers[ans1].pozx * this.cell_width + this.cell_width / 2;
                this.links[index].y1 = (this.map_answers[ans1].pozy + 1) * this.cell_height+halfHeight;
                this.links[index].typeStart=2;
              }else{
                // console.log('case2');
                this.links[index].x1 = this.map_answers[ans1].pozx * this.cell_width + this.cell_width / 2;
                this.links[index].y1 = this.map_answers[ans1].pozy * this.cell_height-halfHeight;            
                this.links[index].typeStart=4;                
              }
              if (LR == "LR"){
                this.CreateArrow(index, "left");
              }else{
                this.CreateArrow(index, "right");
              }

            
      }
     

    },
    CreateArrow(index, from) {
      var p1 = -1;
      var p2 = -1;
      var p3 = -1;
      var p4 = -1;
      var p5 = -1;
      var p6 = -1;
      switch (from) {
        case "left":
          //arrow left
          p1 = this.links[index].x2;
          p2 = this.links[index].y2 - this.triangle_base / 2;
          p3 = this.links[index].x2;
          p4 = this.links[index].y2 + this.triangle_base / 2;
          p5 = this.links[index].x2 + this.triangle_height;
          p6 = this.links[index].y2;

          p1=p1+this.cell_width/2-this.triangle_height;
          p3=p3+this.cell_width/2-this.triangle_height;
          p5=p5+this.cell_width/2-this.triangle_height;
          break;
        case "right":
          //arrow right
          p1 = this.links[index].x2;
          p2 = this.links[index].y2 - this.triangle_base / 2;
          p3 = this.links[index].x2;
          p4 = this.links[index].y2 + this.triangle_base / 2;
          p5 = this.links[index].x2 - this.triangle_height;
          p6 = this.links[index].y2;

          p1=p1-this.cell_width/2+this.triangle_height;
          p3=p3-this.cell_width/2+this.triangle_height;
          p5=p5-this.cell_width/2+this.triangle_height;
          break;
        case "up":
          //arrow up
          p1 = this.links[index].x2 - this.triangle_base / 2;
          p2 = this.links[index].y2;
          p3 = this.links[index].x2 + this.triangle_base / 2;
          p4 = this.links[index].y2;
          p5 = this.links[index].x2;
          p6 = this.links[index].y2 + this.triangle_height;

          p2=p2+this.cell_height/2-this.triangle_height;
          p4=p4+this.cell_height/2-this.triangle_height;
          p6=p6+this.cell_height/2-this.triangle_height;
          break;
        case "down":
          //arrow down
          p1 = this.links[index].x2 - this.triangle_base / 2;
          p2 = this.links[index].y2;
          p3 = this.links[index].x2 + this.triangle_base / 2;
          p4 = this.links[index].y2;
          p5 = this.links[index].x2;
          p6 = this.links[index].y2 - this.triangle_height;

          p2=p2-this.cell_height/2+this.triangle_height;
          p4=p4-this.cell_height/2+this.triangle_height;
          p6=p6-this.cell_height/2+this.triangle_height;

          break;
        default:
          console.log("error in switch");
        // code block
      }
      this.links[index].arrow = {
        p1: p1,
        p2: p2,
        p3: p3,
        p4: p4,
        p5: p5,
        p6: p6
      };
    },
    CheckArrow(ans1, ans2) {
      var gasit = false;
      this.links.forEach(function(link) {
        // console.log(link.start,ans1,link.end,ans2);
        if (
          (link.start == ans1 && link.end == ans2) ||
          (link.start == ans2 && link.end == ans1)
        ) {
          gasit = true;
        }
      });
      // console.log('gasit',gasit);
      if (!gasit) return true;
      return false;
    },
    DeleteAllLinks(cell) {
      var vueObj = this;
      // console.log(cell);
      this.links.forEach(function(link, index, object) {
        // console.log(link.start, link.end, index, vueObj.links[index]);
        if (link.start == cell || link.end == cell) {
          vueObj.links[index].start = -1;
          vueObj.links[index].end = -1;
          vueObj.links[index].display = false;
        }
      });
    },
    ContextMenuCell: function(e, cell) {
      //  console.log(this.map_answers[cell].text);
      if (this.map_answers[cell].text != "") {
        this.DeleteAllLinks(cell);
        this.UnUseQuestion(this.map_answers[cell].text, cell);
        this.ans1=-1;
      } else {
        console.log("context click on a valid cell");
      }
      this.MarkBlock();
      e.preventDefault();
    },
    ContextMenuArrow: function(e, index) {
      this.links[index].start = -1;
      this.links[index].end = -1;
      this.links[index].display = false;
      e.preventDefault();
    },
    UnUseQuestion: function(value, cell) {
      var vueObj = this;
      this.map_questions.forEach(function(question, index, object) {
        // console.log(link.start,link.end, index, vueObj.links[index]);
        if (question.text == value) {
          vueObj.map_questions[index].used = false;
          vueObj.map_answers[cell].text = "";
        }
      });
    },
    MarkBlock(){
        var vueObj=this;
          this.map_answers.forEach(function(ans,index) {
            vueObj.map_answers[index].blocked=false;
          });
          this.map_answers.forEach(function(ans,index) {
            if (ans.text!="") {
              // console.log(ans.text);
              vueObj.map_answers[index].blocked=true;
              vueObj.MarkAround(index);
            }
          });
    },
    MarkAround(index){
      //  A  B  C
      //  D  E  F
      //  G  H  I
      // console.log(this.map_answers[index].pozx);
      
      if (this.map_answers[index].pozy>0){
        //mark B
        this.map_answers[index-this.a_cols].blocked=true;
         //mark A
        if (this.map_answers[index].pozx!=0){
          this.map_answers[index-this.a_cols-1].blocked=true;
        }
        //mark C
        if (this.map_answers[index].pozx<this.a_cols-1){
          this.map_answers[index-this.a_cols+1].blocked=true;
        }
      }  
      if (this.map_answers[index].pozy<this.a_rows-1){
        //mark H
        this.map_answers[index+this.a_cols].blocked=true;
        //mark G
        if (this.map_answers[index].pozx!=0){
          this.map_answers[index+this.a_cols-1].blocked=true;
        }
        //mark I
        if (this.map_answers[index].pozx<this.a_cols-1){
          this.map_answers[index+this.a_cols+1].blocked=true;
        }
      }
      //mark D
      if (this.map_answers[index].pozx!=0){
        this.map_answers[index-1].blocked=true;
      }
      //mark F
      if (this.map_answers[index].pozx<this.a_cols-1){
        this.map_answers[index+1].blocked=true;
      }
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
          this.AddQuestion("C"+this.map_conditions.length);
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
  AddQuestion(name){
    console.log('add question ', name);

     this.map_questions.push({
          x: 0,
          y: 0,
          text: name,
          id: "question_" + name,
          pozx: this.next_Qpozx,
          pozy: this.next_Qpozy,
          used: false,
          poz: this.next_Qpoz,
          hover_text:this.MakeMePretty(this.map_conditions[this.map_conditions.length-1])
        });

    this.GetNextPoz();
    },
    MakeMePretty(cond){
      var cond_pretty="";
      console.log(cond.length);
      cond.forEach(function(condition,index) {
        cond_pretty=cond_pretty+condition.CD1+" "+condition.CD2+" "+condition.CD3+ " "+condition.Next +" "
      });
      // for (var i=0; i<cond.length; i++){
      //   cond_pretty=cond_pretty+cond.CD1+" "+cond.CD2+" "+cond.CD3+ " "+cond.Next+";"  
      // }
      return cond_pretty;
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
</style>
