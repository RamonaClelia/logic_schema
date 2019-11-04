<template>
  <div class="SelectionGridContainer">
    <!-- debug section -->
    <!-- {{q_selected}}..{{cell_selected}}
    <br />
    {{qMoveFrom}}..{{qMoveTo}}
    <br /> 
    <div
      v-for="(link,index) in map_links"
      :key="'link_'+index"
    >{{link.from}} - {{link.to}}..{{link.x1}},{{link.y1}},{{link.x2}},{{link.y2}}..{{link.lbl}}</div>-->
     <!-- {{qMoveFrom}}..{{qMoveTo}} -->
    <!-- questions container -->
    <div>Questions, conditions and terminals:</div>

    <svg :width="QchartWidth" :height="QchartHeight" id="QuestionsContainer">
      <g
        v-for="block in map_array"
        :key="block.id"
        class="rect_question"
        @click="BlockClick(block.poz)"
        v-bind:class="{ qSelected: q_selected==block.poz,qUsed: block.used }"
      >
        <title v-if="!block.used">{{block.hover_text}}</title>
        <rect :x="block.pozx*q_width" :y="block.pozy*q_height" :width="q_width" :height="q_height" />
        <text
          :x="block.pozx*q_width+q_width/2"
          :y="block.pozy*q_height+q_height/2"
          alignment-baseline="middle"
          text-anchor="middle"
          class="text_question"
        >{{block.text}}</text>
        <!-- >{{block.pozx}},{{block.pozy}},{{block.text}}</text> -->
        <!-- >{{block.poz}},{{block.move}},{{block.text}}</text> -->
      </g>
    </svg>

    <!-- conditions container -->
    <div>Conditions builder:</div>
    <div id="Conditions"  v-bind:style="{ width: QchartWidth+'px' }">
    <div
     
      id="ConditionsContainer"
    >
      <select class="DropDownCondition" id="DropDownCondition1" v-model="CD1">
        <option
          v-for="(question,questionIndex) in questions"
          :key="questionIndex"
          :value="question"
        >{{ question }}</option>
      </select>
      <select class="DropDownCondition" id="DropDownCondition1" v-model="CD2">
        <option
          v-for="(operator,operatorIndex) in operators"
          :key="operatorIndex"
          :value="operator"
        >{{ operator }}</option>
      </select>
      <select class="DropDownCondition" id="DropDownCondition1" v-model="CD3">
        <option
          v-for="(answer,answerIndex) in answers"
          :key="answerIndex"
          :value="answer"
        >{{ answer }}</option>
      </select>

      <div class="CondButton" @click="CondClick(1)" v-if="!CB_visible">AND</div>
      <div class="CondButton" @click="CondClick(2)" v-if="!CB_visible">OR</div>
      <div class="CondButton" @click="CondClick(3)" v-if="!CB_visible">DONE</div>
      <div class="CondButton" @click="CondClick(4)" v-if="CB_visible">GENERATE</div>
      <div class="CondButton" @click="CondClick(5)" v-if="CB_visible">DELETE</div>
    </div>

    <!-- temporary conditions display container -->
    <!-- <div
      id="TemporaryConditions"
      v-for="cond in tempCond"
      :key="cond.CD1+cond.CD2+cond.CD3+cond.id"
    >
    {{cond.CD1}} {{cond.CD2}} {{cond.CD3}} {{cond.Next}}
    
    </div> -->
      <div
      id="TemporaryConditions">
      {{MakeMePretty(tempCond)}}
      </div>


    </div>


    <!-- conditions display container -->
    <!-- <div v-for="(condBlock,index1) in map_conditions" :key="index1" class="ConditionDiv">
      C{{index1}}:
      <div
        v-for="(cond,index2) in condBlock"
        :key="index2"
      >{{cond.CD1}} {{cond.CD2}} {{cond.CD3}} {{cond.Next}}</div>
    </div> -->

    <!-- answers container -->
    <div>Answer container:</div>
    <svg :width="AchartWidth" :height="AchartHeight" id="AnswersContainer">
      <g
        v-for="acell in map_answers"
        :key="acell.id"
        @click="CellClick(acell.poz)"
        @contextmenu="ContextMenuQuestionCell($event,acell.poz)"
        
        v-bind:class="{ cellSelected: acell.poz==qMoveFrom||acell.poz==qMoveTo}"
        class="rect_answers"
      >
      <!--  -->
        <rect
          :x="acell.pozx*cell_width"
          :y="acell.pozy*cell_height"
          :width="cell_width"
          :height="cell_height"
          v-bind:class="{ CellBlocked: CellLocked(acell.poz)}"
        />
        <!-- <text :x="acell.pozx*cell_width+cell_width/2" :y="acell.pozy*cell_height+cell_height/2" alignment-baseline="middle" text-anchor="middle">{{acell.poz}}</text> -->
        <title
          v-for="(temp,index) in map_array.filter(item=>{return item.move==acell.poz})"
          :key="temp.id+index"
        >{{temp.hover_text}}</title>

        <text
          v-for="temp in map_array.filter(item=>{return item.move==acell.poz})"
          :key="temp.id"
          :x="acell.pozx*cell_width+cell_width/2"
          :y="acell.pozy*cell_height+cell_height/2"
          alignment-baseline="middle"
          text-anchor="middle"
        >{{temp.text}}</text>
      </g>
      <!-- links -->
      <g
        v-for="(link,index) in map_links"
        :key="'link_'+index"
        @contextmenu="ContextMenuArrow($event,link.from,link.to)" 
        v-bind:class="{ poly_true: link.lbl=='TRUE',poly_false: link.lbl=='FALSE', poly_none: link.lbl=='', menued_link: checkIfMenued(link.from,link.to)}"
        class="poly_group"
      >
        <polyline :points="GetPoints('line',link.x1,link.y1,link.x2,link.y2)" class="poly_line" />
        <polygon :points="GetPoints('arrow',link.x1,link.y1,link.x2,link.y2)" class="poly_arrow" />
      </g>
      <!-- arrow menu group -->
      <g id="ContextMenuArrowGroup" v-if="showArrowMenu">
        <rect
          :x="menuArrowX"
          :y="menuArrowY"
          :width="menu_arrow_width"
          :height="menu_ArrowCell_height*menu_arrow_array.length"
        />
        <text  v-for="(item,index) in menu_arrow_array"
          :key="item" 
          :x="menuArrowX+5" 
          :y="menuArrowY+menu_ArrowCell_height*(index+1)-5"     
          @click="ContextOptionsArrow(index+1)"
          class="menuText"
          >{{item}}
        </text>            
      </g>
      <!-- question menu group -->
      <g id="ContextMenuQuestionGroup" v-if="showQuestionMenu">
        <rect
          :x="menuQuestionX"
          :y="menuQuestionY"
          :width="menu_question_width"
          :height="menu_QuestionCell_height*menu_question_array.length"
        />
        <text  v-for="(item,index) in menu_question_array"
          :key="item" 
          :x="menuQuestionX+5" 
          :y="menuQuestionY+menu_QuestionCell_height*(index+1)-5"     
          @click="ContextOptionsQuestion(index+1)"
          class="menuText"
          >{{item}}
        </text>           

        <!-- <text  v-for="(item,index) in menu_question_array"
          :key="item" 
          :x="menuQuestionX+5" 
          :y="menuQuestionY+menu_QuestionCell_height*(index+1)-5"     
          @click="ContextOptionsQuestion(index+1)"
          class="menuText"
          >{{item}}
        </text>             -->
      </g>
    </svg>
  <!-- Generate Output-->
  <div class="CondButton" @click="GenerareOutput()" >Generate</div>
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
      questions: ["START", "Q1", "Q2", "Q3", "Q4", "Q5", "Q6", "STOP"],
      q_width: 50, //question width
      q_height: 50, //question height
      q_cols: 10, // number of questions per row
      q_rows: -1, //determined based on cols and count
      map_array: [],
      map_answers: [],
      map_conditions: [],
      operators: ["=", "<>", "*","<",">"],
      answers: ["ans1", "ans2", "ans3","40"],
      cell_width: 50, //question width
      cell_height: 50, //question height
      a_cols: 15, //answers_cols
      a_rows: 20, //answers_rows
      q_selected: -1,
      cell_selected: -1,
      nextqx: 0,
      nextqy: 0,
      CD1: "",
      CD2: "",
      CD3: "",
      tempCond: [],
      CB_visible: false,
      qMoveFrom: -1,
      qMoveTo: -1,
      map_links: [],
      arrow_height: 10,
      arrow_base: 20,
      lock_cells: [],
      menuArrowFrom:-1,
      menuArrowTo:-1,
      showArrowMenu:false,
      menuArrowX:0,
      menuArrowY:0,
      menu_arrow_width:75,
      menu_ArrowCell_height:25,
      menu_arrow_array:["Delete","True","False","None","Close"],
      showQuestionMenu:false,
      menuQuestionX:0,
      menuQuestionY:0,
      menu_question_width:75,
      menu_QuestionCell_height:25,
      menu_question_array:["Delete","Close"],
      ContextCellSelected:-1
    };
  },
  created() {
    this.Generate();
    var vueObj = this;

    $(window).resize(function() {
      // console.log("resize");
    });
    // tbd
  },
  watch: {
    showQuestionMenu(newVal){
      if (!newVal){
        this.ContextCellSelected=-1;
      }
    }
  },
  methods: {
    Generate() {
      //add initial questions
      for (var i = 0; i < this.questions.length; i++) {
        this.AddBlock(this.questions[i], "");
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
    AddBlock(qtext, hoverText) {
      //add question or condition in map_array
      var count = this.map_array.length;
      this.map_array.push({
        text: qtext,
        id: "block_" + count,
        pozx: this.nextqx,
        pozy: this.nextqy,
        used: false,
        poz: count,
        hover_text: hoverText,
        move: -1
      });
      if (this.nextqx < this.q_cols - 1) {
        this.nextqx++;
      } else {
        this.nextqx = 0;
        this.nextqy++;
      }

      //get number of rows
      this.q_rows = Math.ceil(this.map_array.length / this.q_cols);
      // rezise question container
      this.QchartWidth = this.q_cols * this.q_width;
      this.QchartHeight = this.q_rows * this.q_height;
    },
    BlockClick(qpoz) {
      this.showArrowMenu=false;
      this.showQuestionMenu=false;
      this.qMoveFrom=-1;
      // console.log('BlockClick for ',qpoz);
      if (!this.map_array[qpoz].used) {
        this.q_selected = qpoz;
      } else {
        // console.log('question used');
      }
    },
    CellClick(poz) {
      this.showArrowMenu=false;
      this.showQuestionMenu=false;
      //check if the cell is locked
      var isLocked = false;
      var vueObj = this;
      this.lock_cells.forEach(function(lock, index) {
        if (lock == poz) {
          isLocked = true;
          vueObj.map_array.forEach(function(obj, index) {
            if (obj.move == poz) {
              isLocked = false;
            }
          });
        }
      });
      if (isLocked) {
        this.q_selected = -1;
        // console.log("just locked");
        return;
      }
      // console.log('selected cell');
      //
      if (this.q_selected != -1) {
        //check if destination was previously selected

        if (!this.CheckEmptyCell(poz)) {
          // console.log('cell already assigned');
          this.q_selected = -1;
          return;
        }
        // console.log('click on ', poz);
        this.cell_selected = poz;
        this.AssignCell();
      } else {
        // console.log('question not selected');
        this.CheckAndMove(poz);
      }
    },
    CheckAndMove(poz) {
      if (this.qMoveFrom == -1 && !this.CheckEmptyCell(poz)) {
        this.qMoveFrom = poz;
        return;
      }
      if (this.qMoveFrom != -1 && this.CheckEmptyCell(poz)) {
        this.qMoveTo = poz;
        var vueObj = this;
        //move arrows
        do {
          var gasit = false;
          this.map_links.forEach(function(link, index) {
            if (link.from == vueObj.qMoveFrom) {
              vueObj.UpdateLinks(vueObj.qMoveTo, link.to, index,link.lbl);
              gasit = true;
            }
            if (link.to == vueObj.qMoveFrom) {
              vueObj.UpdateLinks(link.from, vueObj.qMoveTo, index,link.lbl);
              gasit = true;
            }
          });
        } while (gasit);

        //move question
        this.map_array.forEach(function(obj, index) {
          if (obj.move == vueObj.qMoveFrom) {
            vueObj.map_array[index].move = vueObj.qMoveTo;
            vueObj.qMoveFrom = -1;
            vueObj.qMoveTo = -1;
          }
        });
        this.LockAllCells();
        return;
      }
      // console.log('create link in CheckAndMove');
      if (this.CheckIfLinkPossible(this.qMoveFrom, poz)) {
        // console.log('create new link');
        this.CreateLink(this.qMoveFrom, poz,"");
      } else {
        // console.log("link not possible");
      }
      this.qMoveFrom = -1;
    },
    UpdateLinks(newstart, newend, index,label) {
      this.map_links.splice(index, 1);
      this.CreateLink(newstart, newend,label);
    },
    CheckIfLinkPossible(cell1, cell2) {
      if (cell1 == -1 || cell2 == -1) return false;
      if (cell1 == cell2) return false;
      var possible = true;
      this.map_links.forEach(function(link, index) {
        if (
          (link.from == cell1 && link.to == cell2) ||
          (link.from == cell2 && link.to == cell1)
        ) {
          possible = false;
        }
      });
      return possible;
    },
    CreateLink(start, end,label) {
      // console.log('start, end: ',start, end);
      // console.log(this.a_cols,this.a_rows);
      this.map_links.push({
        from: start,
        to: end,
        x1: this.UpdatePoints(start, end, "startx"),
        y1: this.UpdatePoints(start, end, "starty"),
        x2: this.UpdatePoints(start, end, "endx"),
        y2: this.UpdatePoints(start, end, "endy"),
        lbl: label
      });
    },
    UpdatePoints(start, end, type) {
      var startx = start;
      var starty = 0;
      var endx = end;
      var endy = 0;
      while (startx > this.a_cols - 1) {
        startx = startx - this.a_cols;
        starty++;
      }
      while (endx > this.a_cols - 1) {
        endx = endx - this.a_cols;
        endy++;
      }
      switch (type) {
        case "startx":
          return startx;
        case "starty":
          return starty;
        case "endx":
          return endx;
        case "endy":
          return endy;
      }
    },
    CheckEmptyCell(poz) {
      var isEmpty = true;
      this.map_array.forEach(function(obj, index) {
        if (obj.move == poz) {
          isEmpty = false;
        }
      });

      return isEmpty;
    },
    AssignCell() {
      this.map_array[this.q_selected].used = true;
      this.map_array[this.q_selected].move = this.cell_selected;
      this.q_selected = -1;
      this.cell_selected = -1;
      this.LockAllCells();
    },
    CondClick(type) {
      switch (type) {
        case 1: //case AND
          this.tempCond.push({
            CD1: this.CD1,
            CD2: this.CD2,
            CD3: this.CD3,
            Next: "AND",
            id: this.tempCond.length - 1
          });
          break;
        case 2: //case OR
          this.tempCond.push({
            CD1: this.CD1,
            CD2: this.CD2,
            CD3: this.CD3,
            Next: "OR",
            id: this.tempCond.length - 1
          });
          break;
        case 3: //case DONE
          this.tempCond.push({
            CD1: this.CD1,
            CD2: this.CD2,
            CD3: this.CD3,
            Next: "",
            id: this.tempCond.length - 1
          });
          this.CB_visible = true;
          break;
        case 4: //case GENERATE
          this.map_conditions.push(this.tempCond);
          this.AddBlock(
            "C" + this.map_conditions.length,
            this.MakeMePretty(
              this.map_conditions[this.map_conditions.length - 1]
            )
          );
        case 5: //case GENERATE and DELETE
          this.tempCond = [];
          this.CD1 = "";
          this.CD2 = "";
          this.CD3 = "";
          this.CB_visible = false;
          break;
        default:
          console.log("error in type cond_click");
      }
    },
    MakeMePretty(cond) {
      var cond_pretty = "";
      // console.log(cond.length);
      cond.forEach(function(condition, index) {
        cond_pretty =
          cond_pretty + "("+
          condition.CD1 +
          " " +
          condition.CD2 +
          " " +
          condition.CD3 +
          ") " +
          condition.Next +
          " ";
      });
      // for (var i=0; i<cond.length; i++){
      //   cond_pretty=cond_pretty+cond.CD1+" "+cond.CD2+" "+cond.CD3+ " "+cond.Next+";"
      // }
      return cond_pretty;
    },
    ContextOptionsQuestion(type){
          // console.log('ContextOptionsQuestion');
      var vueObj = this;
      if (type==1){
        //delete arrows
          do {
            var gasit = false;
            this.map_links.forEach(function(link, index) {
              if (link.from == vueObj.ContextCellSelected || link.to == vueObj.ContextCellSelected) {
                vueObj.map_links.splice(index, 1);
                gasit = true;
              }
            });
          } while (gasit);

          //delete cell
          this.map_array.forEach(function(obj, index) {
            if (obj.move == vueObj.ContextCellSelected) {
              vueObj.map_array[index].move = -1;
              vueObj.map_array[index].used = false;
            }
          });
          this.LockAllCells();
          this.qMoveFrom=-1;
          this.qMoveTo=-1;
      }
      // type 2 == close => do nothing
     
      this.showQuestionMenu=false;
    },
    ContextMenuQuestionCell(e, cell) {
      this.q_selected=-1;
      this.qMoveFrom=-1;
      this.qMoveTo=-1;
      this.showArrowMenu=false;
      // console.log('context for ', cell);
      var vueObj = this;
      this.map_array.forEach(function(obj, index) {
        if (obj.move == cell) {
           vueObj.showQuestionMenu=true;
           vueObj.ContextCellSelected=cell;
        }
      });
      this.menuQuestionX=(this.map_answers[cell].pozx+1)*this.cell_width;
      if (this.menuQuestionX+this.menu_question_width-this.cell_width>=this.cell_width*(this.a_cols-1)) this.menuQuestionX=this.map_answers[cell].pozx*this.cell_width-this.menu_question_width;
      
      this.menuQuestionY=this.map_answers[cell].pozy*this.cell_height+this.cell_height/2-this.menu_question_array.length*this.menu_QuestionCell_height/2;

      if (this.menuQuestionY<0) this.menuQuestionY=0;
       if (this.menuQuestionY+this.menu_question_array.length*this.menu_QuestionCell_height>=this.cell_height*(this.a_rows-1)) this.menuQuestionY=this.cell_height*(this.a_rows-1)-this.menu_question_array.length*this.menu_QuestionCell_height+this.cell_height;




      e.preventDefault();
    },
    GetPoints(type, x1, y1, x2, y2) {
      // console.log(type);
      // console.log('Points:', x1,y1,x2,y2);
      var startType = 0;
      var endType = 0;
      // types
      //    2
      // 1     3
      //    4
      var px1 = 0;
      var py1 = 0;
      var px2 = 0;
      var py2 = 0;
      var px3 = 0;
      var py3 = 0;
      var px4 = 0;
      var py4 = 0;
      // console.log('----------------------');
      if (Math.abs(x1 - x2) > 1) {
        if (x1 < x2) {
          startType = 3;
          endType = 1;
        }
        if (x1 > x2) {
          startType = 1;
          endType = 3;
        }
      }
      if (Math.abs(y1 - y2) > 1) {
        if (y1 < y2) {
          startType = 4;
          endType = 2;
        }
        if (y1 > y2) {
          startType = 2;
          endType = 4;
        }
      }
      if (startType == 0) {
        if (y1 < y2) {
          startType = 4;
        }
        if (y1 > y2) {
          startType = 2;
        }
      }
      if (type == "line") {
        px1 = this.returnPoints(startType, 1, x1, y1, x2, y2);
        py1 = this.returnPoints(startType, 2, x1, y1, x2, y2);
        px2 = this.returnPoints(startType, 3, x1, y1, x2, y2);
        py2 = this.returnPoints(startType, 4, x1, y1, x2, y2);

        px3 = this.returnPoints(endType, 3, x2, y2, x1, y1);
        py3 = this.returnPoints(endType, 4, x2, y2, x1, y1);
        px4 = this.returnPoints(endType, 1, x2, y2, x1, y1);
        py4 = this.returnPoints(endType, 2, x2, y2, x1, y1);

        // correction for the arrow space
        switch (endType) {
          case 1:
            px4 = px4 - this.arrow_height;
            break;
          case 2:
            py4 = py4 - this.arrow_height;
            break;
          case 3:
            px4 = px4 + this.arrow_height;
            break;
          case 4:
            py4 = py4 + this.arrow_height;
            break;
        }

        return (
          px1 +
          "," +
          py1 +
          " " +
          px2 +
          "," +
          py2 +
          " " +
          px3 +
          "," +
          py3 +
          " " +
          px4 +
          "," +
          py4
        );
      } else {
        return this.returnArrow(
          endType,
          this.returnPoints(endType, 1, x2, y2, x1, y1),
          this.returnPoints(endType, 2, x2, y2, x1, y1)
        );
      }

      // return "0,40 40,40 40,80 80,80 80,120 120,120 120,160";
    },
    returnArrow(type, x, y) {
      switch (type) {
        case 1:
          return (
            x +
            "," +
            y +
            " " +
            parseInt(x - this.arrow_height) +
            "," +
            parseInt(y + this.arrow_base / 2) +
            " " +
            parseInt(x - this.arrow_height) +
            "," +
            parseInt(y - this.arrow_base / 2)
          );
        case 2:
          return (
            x +
            "," +
            y +
            " " +
            parseInt(x - this.arrow_base / 2) +
            "," +
            parseInt(y - this.arrow_height) +
            " " +
            parseInt(x + this.arrow_base / 2) +
            "," +
            parseInt(y - this.arrow_height)
          );
        case 3:
          return (
            x +
            "," +
            y +
            " " +
            parseInt(x + this.arrow_height) +
            "," +
            parseInt(y + this.arrow_base / 2) +
            " " +
            parseInt(x + this.arrow_height) +
            "," +
            parseInt(y - this.arrow_base / 2)
          );
        case 4:
          return (
            x +
            "," +
            y +
            " " +
            parseInt(x - this.arrow_base / 2) +
            "," +
            parseInt(y + this.arrow_height) +
            " " +
            parseInt(x + this.arrow_base / 2) +
            "," +
            parseInt(y + this.arrow_height)
          );
      }
    },
    returnPoints(type, point, x1, y1, x2, y2) {
      switch (type) {
        case 1:
          if (point == 1) return x1 * this.cell_width;
          if (point == 2)
            return (y1 + 1) * this.cell_height - this.cell_height / 2;
          if (point == 3) return x1 * this.cell_width - this.cell_width / 2;
          if (point == 4)
            return (y1 + 1) * this.cell_height - this.cell_height / 2;
          break;
        case 2:
          if (point == 1)
            return (x1 + 1) * this.cell_width - this.cell_width / 2;
          if (point == 2) return y1 * this.cell_height;
          if (point == 3)
            return (x1 + 1) * this.cell_width - this.cell_width / 2;
          if (point == 4) return y1 * this.cell_height - this.cell_height / 2;
          break;
        case 3:
          if (point == 1) return (x1 + 1) * this.cell_width;
          if (point == 2) return y1 * this.cell_height + this.cell_height / 2;
          if (point == 3)
            return (x1 + 1) * this.cell_width + this.cell_width / 2;
          if (point == 4) return y1 * this.cell_height + this.cell_height / 2;
          break;
        case 4:
          if (point == 1)
            return (x1 + 1) * this.cell_width - this.cell_width / 2;
          if (point == 2) return (y1 + 1) * this.cell_height;
          if (point == 3)
            return (x1 + 1) * this.cell_width - this.cell_width / 2;
          if (point == 4)
            return (y1 + 1) * this.cell_height + this.cell_height / 2;
          break;
      }
    },
    CellLocked(a_cell) {
      var gasit = false;
      this.lock_cells.find(cell => {
        if (cell == a_cell) {
          gasit = true;
        }
      });
      return gasit;
    },
    LockAllCells() {
      // console.log("lock");
      this.lock_cells = [];
      var vueObj = this;
      this.map_array.forEach(function(question, index) {
        if (question.move != -1) {
          //block center
          vueObj.lock_cells.push(question.move);
          //A  B  C
          //D     E
          //F  G  H

          //A
          if (
            vueObj.map_answers[question.move - vueObj.a_cols - 1] &&
            vueObj.map_answers[question.move - vueObj.a_cols - 1].pozx !=
              vueObj.a_cols - 1
          )
            vueObj.lock_cells.push(question.move - vueObj.a_cols - 1);

          //B
          if (vueObj.map_answers[question.move - vueObj.a_cols])
            vueObj.lock_cells.push(question.move - vueObj.a_cols);
          //C
          if (
            vueObj.map_answers[question.move - vueObj.a_cols + 1] &&
            vueObj.map_answers[question.move - vueObj.a_cols + 1].pozx != 0
          )
            vueObj.lock_cells.push(question.move - vueObj.a_cols + 1);
          //D
          if (
            vueObj.map_answers[question.move - 1] &&
            vueObj.map_answers[question.move - 1].pozx != vueObj.a_cols - 1
          )
            vueObj.lock_cells.push(question.move - 1);
          //E
          if (
            vueObj.map_answers[question.move + 1] &&
            vueObj.map_answers[question.move + 1].pozx != 0
          )
            vueObj.lock_cells.push(question.move + 1);
          //F
          if (
            vueObj.map_answers[question.move + vueObj.a_cols - 1] &&
            vueObj.map_answers[question.move + vueObj.a_cols - 1].pozx !=
              vueObj.a_cols - 1
          )
            vueObj.lock_cells.push(question.move + vueObj.a_cols - 1);
          //G
          if (vueObj.map_answers[question.move + vueObj.a_cols])
            vueObj.lock_cells.push(question.move + vueObj.a_cols);
          //H
          if (
            vueObj.map_answers[question.move + vueObj.a_cols + 1] &&
            vueObj.map_answers[question.move + vueObj.a_cols + 1].pozx != 0
          )
            vueObj.lock_cells.push(question.move + vueObj.a_cols + 1);
        }
      });
    },
    ContextMenuArrow(e,from, to){
      this.showArrowMenu=true;
      this.showQuestionMenu=false;
      this.menuArrowFrom=from;
      this.menuArrowTo=to;

      this.menuArrowX=this.cell_width*(this.map_answers[from].pozx+this.map_answers[to].pozx)/2+this.cell_width/2-this.menu_arrow_width/2;

      if (this.menuArrowX<=0) this.menuArrowX=this.cell_width*2/3;
      if (this.menuArrowX+this.menu_arrow_width>=this.cell_width*(this.a_cols-1)) this.menuArrowX=this.cell_width*(this.a_cols)-this.menu_arrow_width-this.cell_width*2/3;

      

      this.menuArrowY=this.cell_height*(this.map_answers[from].pozy+this.map_answers[to].pozy)/2+this.cell_height/2-this.menu_arrow_array.length*this.menu_ArrowCell_height/2;
      if (this.menuArrowY<=0) this.menuArrowY=this.cell_height*2/3;
      if (this.menuArrowY+this.menu_arrow_array.length*this.menu_ArrowCell_height>=this.cell_height*(this.a_rows-1)) this.menuArrowY=this.cell_height*(this.a_rows)-this.menu_arrow_array.length*this.menu_ArrowCell_height-this.cell_height*2/3;
      
      e.preventDefault();

    },
        
    ContextOptionsArrow(type){
    var vueObj = this;
      this.map_links.forEach(function(link, index) {
        if (link.from == vueObj.menuArrowFrom && link.to == vueObj.menuArrowTo) {
          if (type==1) vueObj.map_links.splice(index, 1);
          if (type==2) vueObj.map_links[index].lbl="TRUE";
          if (type==3) vueObj.map_links[index].lbl="FALSE";         
          if (type==4) vueObj.map_links[index].lbl="";
          // type 5 == close => do nothing
        }
      });
      this.showArrowMenu=false;
      this.showQuestionMenu=false;
    },
    checkIfMenued(from, to){
      if (this.menuArrowFrom==from && this.menuArrowTo==to && this.showArrowMenu) return true;
      return false
    },
    GenerareOutput(){
      console.log("-------------------------------------------------------------------");
      console.log('map_array\n');
      console.log(JSON.stringify(this.map_array));
      console.log("-------------------------------------------------------------------");
      console.log('map_links\n');
      console.log(JSON.stringify(this.map_links));
      console.log("-------------------------------------------------------------------");
      console.log('Output\n');
      output="";
      var output=output+"Question||Position||Condition\n";
       this.map_array.forEach(function(question, index) {
        output=output+question.text+"||"+question.move+"||"+question.hover_text+";\n";
      });     
      var output=output+"LinkFrom||LinkTo||LinkLabel\n";
       this.map_links.forEach(function(link, index) {
        output=output+link.from+"||"+link.to+"||"+link.lbl+";\n";
      });  
    console.log(output);
    }
  }
};
</script>
<style>

/* menu container */
#ContextMenuArrowGroup, #ContextMenuQuestionGroup{
  fill:white;
  stroke: black;
}
.menuText{
  fill:black;
  stroke-width: 0;
}
.menuText:hover{
  fill:#3963ff;
  stroke-width: 0;
  font-weight: bold;
}
/* Arrows */
.poly_line {
  stroke-width: 3;
  fill: none;
}
.poly_arrow {
  stroke-width: 3;
  fill: transparent;
}
.poly_true{
  stroke: #24b333;
}
.poly_false{
  stroke: #b32a24;
}
.poly_none{
  stroke: black;
}

.menued_link, .poly_group:hover * {
  stroke-width: 5;
  stroke: #3963ff !important;
}

/* Answer container */

#AnswersContainer {
  border: 1px solid;
  display: flex;
  border-radius: 5px;
}

.rect_answers {
  fill: #ddf2f5;
  stroke-width: 1;
  stroke: #b3b3b3;
  cursor: pointer;
}
.CellBlocked {
  fill: #b6dade;
  cursor: default;
}

.cellSelected {
  fill: #5ca2a9;
}
.cellSelected > .CellBlocked {
  fill: #5ca2a9;
  cursor: default;
}
.rect_answers > text{
  fill: black;
  stroke: none;
  font-weight: bold;
  stroke-width: 0;
  font-size: small;
}
/* Questions container */
#QuestionsContainer {
  border: 1px solid;
  display: flex;
  border-radius: 5px;
}
.rect_question {
  fill: #ddf2f5;
  stroke-width: 1;
  stroke: #777777;
  cursor: pointer;
} 

.text_question {
  fill: black;
  cursor: pointer;
  font-weight: bold;
  stroke-width: 0;
  font-size: small;
}
.qSelected {
  fill: #5ca2a9;
}
.qUsed {
  fill: #777777;
  stroke-width: 1;
  cursor: no-drop;
}
.qUsed > .text_question{
  fill: #777777;
  cursor: no-drop;
}
/* conditions container */
#TemporaryConditions,#ConditionsContainer{
  min-height: 25px;
  display: flex;
}
#ConditionsContainer{
  flex-flow: row;
  height: 50%;
  justify-content: space-evenly;
  padding: 15px 0px;
}
#Conditions{
  /* border:1px solid blue; */
  display: flex; 
  flex-flow: column;
  min-height: 50px;
}
.CondButton {
  border: 1px solid #789a9d;
  min-width: 75px;
  display: inline-flex;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
  border-radius: 5px;
  background-image: linear-gradient(180deg, #7ab3bb,#b0e0e6, #ddf2f5, #ddf2f5,#b0e0e6,#7ab3bb );
  padding: 0px 5px;
   max-width: 200px;
}
.ConditionDiv {
  /* border: 1px solid black; */
  margin-top: 5px;
}
.DropDownCondition{
  min-width: 15%;
      border-radius: 5px;
}
</style>
