<template>
  <div class="row">
    <div class="row justify-content-center w-100">
      <h3 class="col-12">{{$t('inventarios.welcome')}}</h3>
    </div>   
    <div class="col-12 row justify-content-center">
      <base-button type="primary" class="animation-on-hover" @click="visibility.addItem=true; resetModal()">{{$t('common.addItem')}} </base-button>
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
          v-model="nameProduct">
        </base-input>
        <base-input 
          class="text-light ml-3"
          :label="$t('tirillas.unitPrice')"
          type="number"
          :placeholder="$t('tirillas.unitPrice')"
          v-model="priceProduct">
        </base-input>
        <base-input 
          class="text-light ml-3"
          :label="$t('tirillas.quantity')"
          type="number"
          :placeholder="$t('tirillas.quantity')"
          v-model="quantityProduct">
        </base-input>
      </div>
        <div class="ml-auto" slot="footer">  
            <base-button class="animation-on-hover" type="success" @click="addItem()">{{$t('common.addItem')}}</base-button>
            <base-button class="animation-on-hover" type="danger" @click="visibility.addItem=false; ">{{$t('common.cancel')}}</base-button>
        </div>
    </modal>
  </div>
</template>
<script>

import {Modal,BaseAlert} from '@/components'
export default {
  components: {
    BaseAlert,
    Modal
  },
  name: 'notifications',
  
  data() {
    return {
      visibility:{
        addItem : false
      },
      nameProduct     : '',
      priceProduct    : '',
      quantityProduct :  ''
    };
  },
  methods: {
    async addItem(){
      if(this.nameProduct==''||this.priceProduct==''||this.quantityProduct==''){
        this.$swal.fire({
          icon: 'error',
          title: 'Oops...',
          text: 'Llene todos los campos por favor !',
          
        })
      }else{
        try {
          let params={
            product_name  : this.nameProduct,
            price_product : this.priceProduct,
            quantity_product: this.quantityProduct
          }
          this.productsResult = await this.$axios.$post('/product',params)
          this.$swal.fire({  
            icon: 'success',
            title: 'El producto ha sido añadido con exito',
            showConfirmButton: false,
            timer: 1500
          })
          this.visibility.addItem=false
          this.resetModal 
        } catch (error) {
           this.$swal.fire({
            icon: 'error',
            title: 'Oops...',
            text: 'Ya existe un producto con este nombre !'            
          })
          this.visibility.addItem=false
        }
      }
    },
    resetModal(){
      this.nameProduct=''
      this.quantityProduct=''
      this.priceProduct=''
    }
  }

};
</script>
<style>
.card .alert {
  position: relative !important;
  width: 100%;
}
</style>
