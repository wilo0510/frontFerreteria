<template>
  <div class="container">
    <div class="row justify-content-center" v-if="printerSelected==null">
      <h3 class="col-12">{{$t('tirillas.welcome')}}</h3>
    </div>    
    <div class="row justify-content-center" v-if="printerSelected==null">
      <base-input :label="$t('tirillas.selectPrinter')">
        <select class="form-control" 
          v-model   ="printerSelected">
            <option 
            v-for="printer in printers" 
            :key="printer"
            class="text-dark">
              {{printer}}
            </option>
        </select>
      </base-input>
    </div>
    <div class="row justify-content-center" v-if="printerSelected!=null">
      <div class="col-12 row justify-content-center">
        <base-button type="primary" class="mr-2 animation-on-hover" @click="visibility.addItem=true; resetModal()">{{$t('common.addItem')}} </base-button>
        <base-button type="danger" class="mr-2 animation-on-hover" :disabled="selectedItem.length==0" @click="deleteProducts">{{$t('common.deleteItem')}} </base-button>
        <base-button :disabled="items.length==0" @click="processTransaction()">{{$t('common.print')}}</base-button>      
      </div>
      <div class="col-12 row justify-content-center mt-2">
        <p>Total : {{total}}</p>
      </div>
      
    </div>
    <div v-if="printerSelected!=null">
      <el-table
      @selection-change="changeItemsDelete" 
      :data="items"
      height="700">
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>
        <el-table-column 
          prop="quantity"
          :label="$t('tirillas.quantity')"
          width="110">
          <template slot-scope="scope">
            <el-input 
              size="small"
              v-model="scope.row.quantity">

            </el-input>
          </template>
        </el-table-column>
        <el-table-column 
          prop="item"
          :label="$t('tirillas.nameProduct')"
          >
        </el-table-column>
        <el-table-column 
          prop="unit_value"
          :label="$t('tirillas.unitPrice')"
          width="150">
          <template slot-scope="scope">
            <el-input 
              size="small"
              v-model="scope.row.unit_value">

            </el-input>
          </template>
        </el-table-column>
        <el-table-column 
          
          :label="$t('tirillas.total')"
          width="100">
          <template slot-scope="scope">
            
              {{scope.row.unit_value*scope.row.quantity}}
            
          </template>
        </el-table-column>
        <div slot="empty">{{$t('common.empty_data')}}</div>
      </el-table>
      
    </div>
     
    
    <modal
    :show.sync    ="visibility.addItem"
    modalClasses  ="modal-xl"
    gradient      ="default"  
    >
        <div slot="header">
          <h3 class="modal-title  text-light mb-4" id="modal-title-default">{{$t('common.addItem')}}</h3>
          
        </div>
        
        
        <div class="row justify-content-center">
          <base-input 
          class="text-light "
          :label="$t('tirillas.nameProduct')"
          type="text"
          :placeholder="$t('tirillas.nameProduct')"
          v-model="fieldSearch">
          </base-input>
          <base-button 
          class="animation-on-hover ml-3 " 
          type="warning" 
          @click="searchProduct()">
            {{$t('common.search')}}
          </base-button>
        </div>
        <el-table
        @selection-change="handleSelectionChange" 
        :data="productsResult"
        height="250">
          <el-table-column
            type="selection"
            width="55">
          </el-table-column>
          <el-table-column
            v-for="column in tableColumns"
            :key="column.label"
            v-bind="column"
            >
          </el-table-column>
          <div slot="empty">{{$t('common.empty_data')}}</div>
        </el-table>
        <div class="ml-auto" slot="footer">  
            <p v-if="selectedSearchProduct.length!=0">{{selectedSearchProduct.length}} items seleccionados</p>        
            <base-button class="animation-on-hover" type="success" :disabled="selectedSearchProduct.length==0" @click="addItem()">{{$t('common.addItem')}}</base-button>
            <base-button class="animation-on-hover" type="danger" @click="visibility.addItem=false; ">{{$t('common.cancel')}}</base-button>
                    
        </div>
        
    </modal>
    
  </div>
  
</template>
<script>
import JSPM from "jsprintmanager";
import LineChart from '@/components/Charts/LineChart';
import BarChart from '@/components/Charts/BarChart';
import TaskList from '@/components/Dashboard/TaskList';
import {Modal,BaseAlert} from '@/components'
import { Table, TableColumn , Radio} from 'element-ui';

export default {
  name: 'tirillas',
  components: {
    BaseAlert,
    Modal,
    LineChart,
    BarChart,
    TaskList,
    [Table.name]: Table,
    [TableColumn.name]: TableColumn,
    [Radio.name]: Radio,
  },
  data () {
    return {
      items           : [],
      selectedSearchProduct : [],      
      productsResult  : [],
      fieldSearch     : '',
      printerSelected : null,
      printers        : [],
      selectedItem    : [],
      
      tableColumns    : [             //nombre de las columnas de la tabla
          {
          prop        :   'nameproduct',
          label       :   this.$t('tirillas.nameProduct'),
          minWidth    :   300,
          sortable    :   false  
          },
          {
          prop        :   'unitprice',
          label       :   this.$t('tirillas.unitPrice'),
          minWidth    :   150,
          sortable    :   false
          },
          {
          prop        :   'quantity',
          label       :   this.$t('tirillas.quantity'),
          minWidth    :   130,
          sortable    :   false
          },
         
      ],
      visibility      : {
        addItem : false
      }
    };
  },
  computed: {
     
    total(){
      return this.items.reduce(function(a,c){
        return a +  Number((c.quantity*c.unit_value) || 0)
      },0)
    }
  },
  methods: {
    deleteProducts(){
      this.$swal.fire({
        title: 'Realmente desea eliminar estos productos?',
        showDenyButton: true,        
        confirmButtonText: `Cancelar`,
        denyButtonText: `Eliminar`,
      }).then((result) => {
        /* Read more about isConfirmed, isDenied below */
       if (result.isDenied) {
         let position=0
         for(var i=0;i<this.selectedItem.length;i++){
            position=this.getPosicionArreglo(this.selectedItem[i].id,this.items)
            console.log(position)
            this.items.splice(position,1)
         }
          
        }
      })
    },
    changeItemsDelete(val){
      this.selectedItem = val;
    },
    processTransaction(){
      this.print()
      this.cleanScreen()
    },
    cleanScreen(){
      this.items=[]
    },
    print(){
        //Create a ClientPrintJob
        var cpj = new JSPM.ClientPrintJob();
        //Set Printer type (Refer to the help, there many of them!)
        
        cpj.clientPrinter = new JSPM.InstalledPrinter(this.printerSelected);
        
        //Set content to print...
        //Create ESP/POS commands for sample label
        var esc           = '\x1B'; //ESC byte in hex notation
        var newLine       = '\x0A'; //LF byte in hex notation
        var horizontalTab = '\x09'; //horizontal tab

        var cmds = esc + "@"; //Initializes the printer (ESC @)
        cmds  +=  esc + '!' + '\x38'; //Emphasized + Double-height + Double-width mode selected (ESC ! (8 + 16 + 32)) 56 dec => 38 hex
        cmds  +=  '  FERRELECTRICOS DISMEC'; //text to print
        cmds  +=  newLine + newLine;
        cmds  +=  esc + '!' + '\x18'; //Emphasized + Double-height mode selected (ESC ! (16 + 8)) 24 dec => 18 hex
        cmds  +=  horizontalTab
        cmds  +=  '     Tel: 5264732-3007909776'
        cmds  +=  newLine + newLine;
        cmds  +=  horizontalTab + horizontalTab
        cmds  +=  `   ${this.actualDate}`
        cmds  +=  esc + '!' + '\x00'; //Character font A selected (ESC ! 0)
        cmds  +=  newLine + newLine;   
        cmds  +=  'Cant'
        cmds  +=  horizontalTab + horizontalTab
        cmds  +=  'Nom prod'
        cmds  +=  horizontalTab + horizontalTab + horizontalTab
        cmds  +=  'Total'
        cmds  +=  newLine + newLine;        
        for(var i=0;i<this.items.length;i++){
          cmds  +=  `${this.items[i].quantity}   `          
          cmds  +=  `${this.items[i].item}   ` 
          cmds  +=  this.items[i].quantity*this.items[i].unit_value
          cmds  +=  newLine + newLine;
        }
        cmds  += esc + '!' + '\x18'; //Emphasized + Double-height mode selected (ESC ! (16 + 8)) 24 dec => 18 hex
        cmds  +=  newLine + newLine; 
        cmds  +=  horizontalTab + horizontalTab + horizontalTab
        cmds  +=  'Total a pagar' 
        cmds  +=  horizontalTab
        cmds  +=  this.total
        cmds  +=  newLine + newLine;
        cmds  +=  newLine + newLine;
        cmds  +=  newLine + newLine;
        cmds  +=  esc + "m"
        // cmds += 'COOKIES                   5.00'; 
        // cmds += newLine;
        // cmds += 'MILK 65 Fl oz             3.78';
        // cmds += newLine + newLine;
        // cmds += 'SUBTOTAL                  8.78';
        // cmds += newLine;
        // cmds += 'TAX 5%                    0.44';
        // cmds += newLine;
        // cmds += 'TOTAL                     9.22';
        // cmds += newLine;
        // cmds += 'CASH TEND                10.00';
        // cmds += newLine;
        // cmds += 'CASH DUE                  0.78';
        // cmds += newLine + newLine;
        // cmds += esc + '!' + '\x18'; //Emphasized + Double-height mode selected (ESC ! (16 + 8)) 24 dec => 18 hex
        // cmds += '# ITEMS SOLD 2';
        // cmds += esc + '!' + '\x00'; //Character font A selected (ESC ! 0)
        // cmds += newLine + newLine;
        // cmds += '11/03/13  19:53:17';

        cpj.printerCommands = cmds;
        //Send print job to printer!
        cpj.sendToClient();
      // try {
      //   let response= await this.$axios.post('/receipt')
      //   console.log(response)
      // } catch (error) {
      //   console.log(error)
      // }
      
      
    },
    addItem(){      
      if(this.selectedSearchProduct.length!=0){
        let position=0
        for(var i=0;i<this.selectedSearchProduct.length;i++){
          position= this.getPosicionArreglo(this.selectedSearchProduct[i].id,this.items)
          if(position!=-1){
            this.$swal.fire({
             icon: 'error',
             title: 'Oops...',
             text: `${this.selectedSearchProduct[i].nameproduct} ya seleccionado, cambia la cantidad!`
            })
          }else{
            let newItem={
              id          : this.selectedSearchProduct[i].id,
              quantity    : 1,
              item        : this.selectedSearchProduct[i].nameproduct,
              unit_value  : this.selectedSearchProduct[i].unitprice          
            }
            this.items.push(newItem)
          }
        }
        this.visibility.addItem=false
        this.resetModal
       
        
      }else{
        bvModalEvt.preventDefault()
        this.$swal.fire({
          icon: 'error',
          title: 'Oops...',
          text: 'Por favor selecciona un articulo!'
        })
      }
    },
    //METODO QUE ENCUENTRA EL INDICE DENTRO DE UN ARREGLO DE OBJETOS
    getPosicionArreglo(itemSearch,array){
      return array.map(function(e){
                            return e.id
                        }).indexOf(itemSearch)
    },
    handleSelectionChange(val) {
      this.selectedSearchProduct = val;
    },
    async searchProduct(){
      try {
        if(this.fieldSearch!=''){
          let params={
            product_name  : this.fieldSearch
          }
          this.productsResult = await this.$axios.$get('/product',{params})          
        }
        else{
          this.$swal.fire({
            icon:   'error',
            title:  'Oops...',
            text:   'Ingrese el nombre del producto',
          })
        }
      } catch (error) {
        console.log(error)
      }
      
    },
    onInit(){
      var _this = this;
      JSPM.JSPrintManager.auto_reconnect = true;
      JSPM.JSPrintManager.start();
      JSPM.JSPrintManager.WS.onStatusChanged = function () {
        _this.getPrinters().then((p)=>{
          _this.printers = p;
          _this.$nextTick();
        });
      };
    },
    getActualDate(){
      this.actualDate=new Date(Date.now()).toLocaleDateString(['ban', 'id'])      
    },
    getPrinters(){
      return new Promise((ok, err) => {
        let printers = [];
        if(JSPM.JSPrintManager.websocket_status == JSPM.WSStatus.Open) {
          JSPM.JSPrintManager.getPrinters().then(function (myPrinters) {
            printers = myPrinters;           
            ok(printers);
          }).catch((e)=>err(e));
        } else { console.warn("JSPM WS not open"); ok(printers); }
      });       
    },
    resetModal(){
      this.fieldSearch    = '',
      this.productsResult = [],
      this.selectedSearchProduct   = []
    },
  },
  mounted () {
    this.onInit(),
    this.getActualDate()
  }
}
</script>
<style >
*{
  font-size: 105%;
}
</style>
