<template>
    <v-layout align-start>
        <v-flex>
            <v-toolbar flat color="white">
                <v-toolbar-title>Ventas</v-toolbar-title>
                <v-divider
                class="mx-2"
                inset
                vertical
                ></v-divider>
                <v-spacer></v-spacer>
                <v-text-field v-if="verNuevo==0" class="text-xs-center" v-model="search" append-icon="search" 
                label="Búsqueda" single-line hide-details></v-text-field>
                <v-spacer></v-spacer>
                <v-btn color="primary" v-if="verNuevo==0" @click="mostrarNuevo()" dark class="mb-2">Nuevo</v-btn>
                <v-dialog v-model="dialog" max-width="1000px">
                    <v-card>
                        <v-card-title> 
                            <span class="headline">Seleccione un artículo</span>
                        </v-card-title>            
                        <v-card-text>
                            <v-container grid-list-md>
                                <v-layout wrap>
                                    <v-flex xs12 sm12 md12 lg12 xl12>                                        
                                        <v-text-field v-model="texto" @keyup.enter="listarArticulos()" class="text-xs-center" append-icon="search" 
                                        label="Búsqueda"></v-text-field>
                                        <template>
                                            <v-data-table
                                                :headers="cabeceraArticulos"
                                                :items="articulos"
                                                class="elevation-1"
                                            >
                                                <template v-slot:items="props">
                                                <td class="justify-center layout px-0">
                                                    <v-icon
                                                    small
                                                    class="mr-2"
                                                    @click="agregarDetalle(props.item)"
                                                    >
                                                    add
                                                    </v-icon>                                                    
                                                </td>
                                                <td>{{ props.item.codigo }}</td>
                                                <td>{{ props.item.nombre }}</td>
                                                <td>{{ props.item.categoria.nombre }}</td>
                                                <td>{{ props.item.stock }}</td>
                                                <td>{{ props.item.precio_venta }}</td>
                                                <td>{{ props.item.descripcion }}</td>
                                                <td>
                                                    <div v-if="props.item.estado">
                                                        <span class="blue--text">Activo</span>
                                                    </div>
                                                    <div v-else>
                                                        <span class="red--text">Inactivo</span>
                                                    </div>
                                                </td>                
                                                </template>
                                            </v-data-table>
                                        </template>
                                    </v-flex>
                                </v-layout>
                            </v-container>
                        </v-card-text>            
                        <v-card-actions>
                        <v-spacer></v-spacer>
                            <v-btn color="blue darken-1" flat @click="close">Cancelar</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="adModal" max-width="290">
                    <v-card>
                        <v-card-title class="headline" v-if="adAccion==1">
                            Activar Item
                        </v-card-title>
                        <v-card-title class="headline" v-if="adAccion==2">
                            Desactivar Item
                        </v-card-title>
                        <v-card-text>
                            Estás a punto de <span v-if="adAccion==1">activar </span>
                            <span v-if="adAccion==2">desactivar </span> el item {{adNombre}}
                        </v-card-text>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn @click="activarDesactivarCerrar()" color="green darken-1" flat="flat">
                                Cancelar
                            </v-btn>
                            <v-btn v-if="adAccion==1" @click="activar()" color="orange darken-4" flat="flat">
                                Activar
                            </v-btn>
                            <v-btn v-if="adAccion==2" @click="desactivar()" color="orange darken-4" flat="flat">
                                Desactivar
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="comprobanteModal" max-width="1000px">
                    <v-card>
                        <v-card-title class="headline">
                            <v-btn @click="crearPDF()"><v-icon>print</v-icon></v-btn> Reporte de venta
                        </v-card-title>

                        <v-card-text>
                            <div id="factura">
                                <header>
                                    <div id="logo">
                                        <img src="data:image/png;base64,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" id="imagen">
                                    </div>
                                    <div id="datos">
                                        <p id="encabezado">
                                            <b>IncanatoIT</b>
                                            <br>José Gálvez 1368, Chongoyape - Chiclayo, Perú<br>Telefono:(+51)931742904<br>Email:jcarlos.ad7@gmail.com
                                        </p>
                                    </div>
                                    <div id="fact">
                                        <p>{{tipo_comprobante}}<br>
                                        {{serie_comprobante}}-{{num_comprobante}}<br>
                                        {{fecha}}</p>
                                    </div>
                                </header>
                                <br>
                                <section>
                                    <div>
                                        <table id="facliente">
                                            <tbody>
                                                <tr>
                                                    <td id="cliente">
                                                        <strong>Sr(a). {{persona.nombre}}</strong><br>
                                                        <strong>Documento:</strong> {{persona.num_documento}}<br>
                                                        <strong>Dirección:</strong> {{persona.direccion}}<br>
                                                        <strong>Teléfono:</strong> {{persona.telefono}}<br>
                                                        <strong>Email:</strong> {{persona.email}}
                                                    </td>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </section>
                                <br>
                                <section>
                                    <div>
                                        <table id="facarticulo">
                                            <thead>
                                                <tr id="fa">
                                                    <th>CANT</th>
                                                    <th>DESCRIPCION</th>
                                                    <th>PRECIO UNIT</th>
                                                    <th>DESC.</th>
                                                    <th>PRECIO TOTAL</th>
                                                </tr>
                                            </thead>
                                            <tbody>
                                                <tr v-for="det in detalles" :key="det._id">
                                                    <td style="text-align:center;">{{det.cantidad}}</td>
                                                    <td>{{det.articulo}}</td>
                                                    <td style="text-align:right;">{{det.precio}}</td>
                                                    <td style="text-align:right;">{{det.descuento}}</td>
                                                    <td style="text-align:right;">{{(det.cantidad*det.precio)-det.descuento}}</td>
                                                </tr>
                                            </tbody>
                                            <tfoot>
                                                <tr>
                                                    <th></th>
                                                    <th></th>
                                                    <th></th>
                                                    <th style="text-align:right;">SUBTOTAL</th>
                                                    <th style="text-align:right;">$ {{totalParcial=(total-totalImpuesto).toFixed(2)}}</th>
                                                </tr>
                                                <tr>
                                                    <th></th>
                                                    <th></th>
                                                    <th></th>
                                                    <th style="text-align:right;">IVA({{impuesto}}%)</th>
                                                    <th style="text-align:right;">$ {{totalImpuesto=((total*impuesto)/(1+impuesto)).toFixed(2)}}</th>
                                                </tr>
                                                <tr>
                                                    <th></th>
                                                    <th></th>
                                                    <th></th>
                                                    <th style="text-align:right;">TOTAL</th>
                                                    <th style="text-align:right;">$ {{total=calcularTotal}}</th>
                                                </tr>
                                            </tfoot>
                                        </table>
                                    </div>
                                </section>
                                <br>
                                <br>
                                <footer>
                                    <div id="gracias">
                                        <p><b>Gracias por su compra!</b></p>
                                    </div>
                                </footer>
                            </div>
                        </v-card-text>

                        <v-card-actions>
                            <v-btn @click="ocultarComprobante()" color="blue darken-1" flat>Cancelar</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </v-toolbar>
            <v-data-table
                :headers="headers"
                :items="ventas"
                :search="search"
                class="elevation-1"
                v-if="verNuevo==0"
            >
                <template v-slot:items="props">
                    <td class="justify-center layout px-0">
                        <v-icon small class="mr-2" @click="verIngreso(props.item)">
                            tab
                        </v-icon>
                        <v-icon small class="mr-2" @click="mostrarComprobante(props.item)">
                            print
                        </v-icon>
                        <template v-if="props.item.estado">
                            <v-icon
                            small
                            @click="activarDesactivarMostrar(2,props.item)"
                            >
                            block
                            </v-icon>
                        </template>
                        <template v-else>
                            <v-icon
                            small
                            @click="activarDesactivarMostrar(1,props.item)"
                            >
                            check
                            </v-icon>
                        </template>
                    </td>
                    <td>{{ props.item.usuario.nombre }}</td>
                    <td>{{ props.item.persona.nombre }}</td>
                    <td>{{ props.item.tipo_comprobante }}</td>
                    <td>{{ props.item.serie_comprobante }}</td>
                    <td>{{ props.item.num_comprobante }}</td>
                    <td>{{ props.item.createdAt }}</td>
                    <td>{{ props.item.impuesto }}</td>
                    <td>{{ props.item.total }}</td>
                    <td>
                        <div v-if="props.item.estado">
                            <span class="blue--text">Aceptado</span>
                        </div>
                        <div v-else>
                            <span class="red--text">Anulado</span>
                        </div>
                    </td>                
                    </template>
                    <template v-slot:no-data>
                    <v-btn color="primary" @click="listar()">Resetear</v-btn>
                </template>
            </v-data-table>
            <v-container grid-list-sm class="pa-4 white" v-if="verNuevo">
                <v-layout row wrap>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-select v-model="tipo_comprobante" 
                        :items="comprobantes" label="Tipo Comprobante">
                        </v-select>
                    </v-flex>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-text-field v-model="serie_comprobante" label="Serie Comprobante">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-text-field v-model="num_comprobante" label="Número Comprobante">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm8 md8 lg8 xl8>
                        <v-autocomplete :items="personas" v-model="persona" label="Cliente">
                        </v-autocomplete>
                    </v-flex>
                    <v-flex xs12 sm4 md4 lg4 xl4>
                        <v-text-field type="number" v-model="impuesto" label="Impuesto">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm8 md8 lg8 x8>
                        <v-text-field v-model="codigo" label="Código" @keyup.enter="buscarCodigo()">
                        </v-text-field>
                    </v-flex>
                    <v-flex xs12 sm2 md2 lg2 xl2>
                        <v-btn small fab dark color="teal" @click="mostrarModalArticulos()">
                            <v-icon dark>list</v-icon>
                        </v-btn>
                    </v-flex>
                    <v-flex xs12 sm2 md2 lg2 xl2 v-show="errorArticulo">
                        <div class="red--text" v-text="errorArticulo">

                        </div>
                    </v-flex>
                    <v-flex xs12 sm12 md12 lg12 xl12>
                        <template>
                            <v-data-table
                                :headers="cabeceraDetalles"
                                :items="detalles"
                                hide-actions
                                class="elevation-1"
                            >
                                <template slot="items" slot-scope="props">
                                    <td>
                                        <v-icon
                                        small
                                        class="mr-2"
                                        @click="eliminarDetalle(detalles,props.item)"
                                        >
                                        delete
                                        </v-icon>
                                    </td>
                                    <td class="text-xs-center">{{ props.item.articulo }}</td>
                                    <td class="text-xs-center"><v-text-field v-model="props.item.cantidad" type="number"></v-text-field></td>
                                    <td class="text-xs-center"><v-text-field v-model="props.item.precio" type="number"></v-text-field></td>
                                    <td class="text-xs-center"><v-text-field v-model="props.item.descuento" type="number"></v-text-field></td>
                                    <td class="text-xs-right">$ {{ (props.item.cantidad * props.item.precio)-props.item.descuento}}</td>
                                </template>
                                <template slot="no-data">
                                    <h3>No hay artículos agregados al detalle.</h3>
                                </template>
                            </v-data-table>
                            <v-flex class="text-xs-right">
                                <strong>Total Parcial:</strong> $ 
                                {{totalParcial=(total-totalImpuesto).toFixed(2)}}
                            </v-flex>
                            <v-flex class="text-xs-right">
                                <strong>Total Impuesto:</strong> $ 
                                {{totalImpuesto=((total*impuesto)/(1+impuesto)).toFixed(2)}}
                            </v-flex>
                            <v-flex class="text-xs-right">
                                <strong>Total Neto:</strong> $ {{total=calcularTotal}}
                            </v-flex>
                        </template>
                    </v-flex>
                    <v-flex xs12 sm12 md12 v-show="valida">
                        <div class="red--text" v-for="v in validaMensaje" :key="v" v-text="v">

                        </div>
                    </v-flex>                    
                    <v-flex xs12 sm12 md12 lg12 xl12>
                        <v-btn color="blue darken-1" flat @click.native="ocultarNuevo()">Cancelar</v-btn>
                        <v-btn color="success" v-if="verDetalle==0" @click.native="guardar()">Guardar</v-btn>
                    </v-flex>
		        </v-layout>
            </v-container>
        </v-flex>
    </v-layout>
</template>
<script>
    import axios from 'axios'
    import jsPDF from 'jspdf'
    import html2canvas from 'html2canvas';
    export default {
        data(){
            return{
                dialog: false,
                search:'',
                ventas:[],
                headers: [
                    { text: 'Opciones', value: 'opciones', sortable: false },
                    { text: 'Usuario', value: 'usuario.nombre', sortable: true },
                    { text: 'Cliente', value: 'persona.nombre', sortable: true },
                    { text: 'Tipo Comprobante', value: 'tipo_comprobante', sortable: true },
                    { text: 'Serie comprobante', value: 'serie_comprobante', sortable: false  },
                    { text: 'Número comprobante', value: 'num_comprobante', sortable: false  },
                    { text: 'Fecha', value: 'createdAt', sortable: false  },
                    { text: 'Impuesto', value: 'impuesto', sortable: false  },
                    { text: 'Total', value: 'total', sortable: false  },
                    { text: 'Estado', value: 'estado', sortable: false  }                
                ],
                _id:'',
                persona:'',
                personas:[],
                tipo_comprobante:'',
                comprobantes: ['BOLETA','FACTURA','TICKET','GUIA'],
                serie_comprobante:'',
                num_comprobante: '',
                impuesto: 0.18,
                codigo:'',
                cabeceraDetalles:[
                    { text: 'Borrar', value: 'borrar', sortable: false },
                    { text: 'Artículo', value: 'articulo', sortable: false },
                    { text: 'Cantidad', value: 'cantidad', sortable: false },
                    { text: 'Precio', value: 'precio', sortable: false },
                    { text: 'Descuento', value: 'descuento', sortable: false },
                    { text: 'Sub Total', value: 'subtotal', sortable: false  }
                ],
                detalles:[],
                verNuevo:0,
                errorArticulo:null,
                total:0,
                totalParcial:0,
                totalImpuesto:0,
                articulos:[],
                texto:'',
                cabeceraArticulos: [
                    { text: 'Seleccionar', value: 'seleccionar', sortable: false },
                    { text: 'Código',value: 'codigo', sortable: false},
                    { text: 'Nombre',value: 'nombre', sortable: true},
                    { text: 'Categoría',value: 'categoria.nombre', sortable: true},
                    { text: 'Stock',value: 'stock', sortable: false},
                    { text: 'Precio Venta',value: 'precio_venta', sortable: false},
                    { text: 'Descripción', value: 'descripcion', sortable: false },              
                    { text: 'Estado', value: 'estado', sortable: false }
                ],
                verDetalle:0,
                valida:0,
                validaMensaje:[],
                adModal:0,
                adAccion:0,
                adNombre:'',
                adId:'',
                comprobanteModal:0,
                fecha:null
            }
        },
        computed: {
            calcularTotal: function(){
                let resultado=0.0;
                for(var i=0;i<this.detalles.length;i++){
                    resultado=resultado+((this.detalles[i].cantidad*this.detalles[i].precio)-this.detalles[i].descuento);
                }
                return resultado;
            }
        },
        watch: {
            dialog (val) {
            val || this.close()
            }
        },
        created () {
            this.listar();
            this.selectPersona();
        },
        methods: {
            crearPDF(){
                var quotes = document.getElementById('factura');
                html2canvas(quotes).then(function (canvas) {
                    var imgData = canvas.toDataURL('image/png');
                    var imgWidth = 210;
                    var pageHeight = 295;

                    var imgHeight = canvas.height * imgWidth / canvas.width;
                    var heightLeft = imgHeight;
                    
                    var doc = new jsPDF('p', 'mm', 'a4');
                    var position = 0;

                    doc.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
                    heightLeft -= pageHeight;

                    while (heightLeft >= 0) {
                        position = heightLeft - imgHeight;
                        doc.addPage();
                        doc.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
                        heightLeft -= pageHeight;
                    }
                    doc.save('ComprobanteVenta.pdf');
                });
            },
            mostrarComprobante(item){
                this.limpiar();
                this.tipo_comprobante=item.tipo_comprobante;
                this.serie_comprobante=item.serie_comprobante;
                this.num_comprobante=item.num_comprobante;
                this.fecha=item.createdAt;
                this.persona=item.persona;
                this.impuesto=item.impuesto;
                this.listarDetalles(item._id);
                this.comprobanteModal=1;
            },
            ocultarComprobante(){
                this.comprobanteModal=0;
            },
            selectPersona(){
                let me=this;
                let personaArray=[];
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('persona/listClientes',configuracion).then(function (response){
                    personaArray=response.data;
                    personaArray.map(function(x){
                        me.personas.push({text:x.nombre, value:x._id});
                    });
                }).catch(function(error){
                    console.log(error);
                });
            },
            buscarCodigo(){
                let me=this;
                me.errorArticulo=null;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('articulo/queryCodigo?codigo='+this.codigo,configuracion).then(function (response){
                    me.agregarDetalle(response.data);
                }).catch(function(error){
                    me.errorArticulo='No existe el artículo.';
                });

            },
            agregarDetalle(data){
                this.errorArticulo=null;
                if (this.encuentra(data._id)==true){
                    this.errorArticulo='El artículo ya ha sido agregado.';
                }
                else{
                    this.detalles.push(
                        {
                            _id:data._id,
                            articulo:data.nombre,
                            cantidad:1,
                            precio:data.precio_venta,
                            descuento:0
                        }
                    );
                    this.codigo='';
                }
                
            },
            encuentra(id){
                let sw=0;
                for (var i=0;i<this.detalles.length;i++){
                    if(this.detalles[i]._id==id){
                        sw=true;
                    }
                }
                return sw;
            },
            eliminarDetalle(arr,item){
                let i=arr.indexOf(item);
                if (i!= -1){
                    arr.splice(i,1);
                }
            },
            listarArticulos(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('articulo/list?valor='+this.texto,configuracion).then(function (response){
                    me.articulos=response.data;
                }).catch(function(error){
                    console.log(error);
                });
            },
            mostrarModalArticulos(){
                this.dialog=1;
            },
            listarDetalles(id){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('venta/query?_id='+id,configuracion).then(function (response){
                    me.detalles=response.data.detalles;
                }).catch(function(error){
                    console.log(error);
                });
            },
            verIngreso(item){
                this.limpiar();
                this.tipo_comprobante=item.tipo_comprobante;
                this.serie_comprobante=item.serie_comprobante;
                this.num_comprobante=item.num_comprobante;
                this.persona=item.persona._id;
                this.impuesto=item.impuesto;
                this.listarDetalles(item._id);
                this.verNuevo=1;
                this.verDetalle=1;
            },
            listar(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};            
                axios.get('venta/list',configuracion).then(function (response){
                    me.ventas=response.data;
                }).catch(function(error){
                    console.log(error);
                });

            },
            limpiar(){
                this._id='';
                this.tipo_comprobante='';
                this.serie_comprobante='';
                this.num_comprobante='';
                this.impuesto=0.18;
                this.codigo='';
                this.total=0;
                this.totalParcial=0;
                this.totalImpuesto=0;
                this.detalles=[];
                this.verNuevo=0;
                this.valida=0;
                this.validaMensaje=[];
                this.verDetalle=0;
            },
            validar(){
                this.valida=0;
                this.validaMensaje=[];
                if(!this.persona){
                    this.validaMensaje.push('Seleccione un cliente.');
                }
                if(!this.tipo_comprobante){
                    this.validaMensaje.push('Seleccione un tipo de comprobante.');
                }
                if(!this.num_comprobante){
                    this.validaMensaje.push('Ingrese el número del comprobante.');
                }
                if(!this.impuesto || this.impuesto<0 || this.impuesto>1){
                    this.validaMensaje.push('Ingrese un impuesto válido.');
                }
                if(this.detalles.length<=0){
                    this.validaMensaje.push('Ingrese al menos un artículo al detalle');
                }
                if (this.validaMensaje.length){
                    this.valida=1;
                }
                return this.valida;
            },
            mostrarNuevo(){
                this.verNuevo=1;
            },
            ocultarNuevo(){
                this.verNuevo=0;
                this.limpiar();
            },
            guardar(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};
                if (this.validar()){
                    return;
                }
                //Código para guardar
                axios.post('venta/add',
                {
                    'persona':this.persona,
                    'usuario':this.$store.state.usuario._id,
                    'tipo_comprobante': this.tipo_comprobante,
                    'serie_comprobante': this.serie_comprobante,
                    'num_comprobante':this.num_comprobante,
                    'impuesto': this.impuesto,
                    'total':this.total,
                    'detalles':this.detalles
                },configuracion)
                .then(function(response){
                    me.limpiar();
                    me.close();
                    me.listar();
                })
                .catch(function(error){
                    console.log(error);
                });
            },
            activarDesactivarMostrar(accion,item){
                this.adModal=1;
                this.adNombre=item.num_comprobante;
                this.adId=item._id;
                if (accion==1){
                    this.adAccion=1;
                } else if(accion==2){
                    this.adAccion=2;
                } else{
                    this.adModal=0;
                }
            },
            activarDesactivarCerrar(){
                this.adModal=0;
            },
            activar(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};
                axios.put('venta/activate',{'_id':this.adId},configuracion)
                    .then(function(response){
                        me.adModal=0;
                        me.adAccion=0;
                        me.adNombre='';
                        me.adId='';
                        me.listar();
                    })
                    .catch(function(error){
                        console.log(error);
                    });
            },
            desactivar(){
                let me=this;
                let header={"Token" : this.$store.state.token};
                let configuracion= {headers : header};
                axios.put('venta/deactivate',{'_id':this.adId},configuracion)
                    .then(function(response){
                        me.adModal=0;
                        me.adAccion=0;
                        me.adNombre='';
                        me.adId='';
                        me.listar();
                    })
                    .catch(function(error){
                        console.log(error);
                    });
            },
            close () {
                this.dialog = false;
            }
        }
    }
</script>
<style>
	    #factura {
            padding: 20px;
            font-family: Arial, sans-serif;
            font-size: 16px ;
        }

        #logo {
            float: left;
            margin-left: 2%;
            margin-right: 2%;
        }
        #imagen {
            width: 100px;
        }

        #fact {
            font-size: 18px;
            font-weight: bold;
            text-align: center;
        }   

        #datos {
            float: left;
            margin-top: 0%;
            margin-left: 2%;
            margin-right: 2%;
            /*text-align: justify;*/
        }

        #encabezado {
            text-align: center;
            margin-left: 10px;
            margin-right: 10px;
            font-size: 16px;
        }

        section {
            clear: left;
        }

        #cliente {
            text-align: left;
        }

        #facliente {
            width: 40%;
            border-collapse: collapse;
            border-spacing: 0;
            margin-bottom: 15px;
        }

        #fa {
            color: #FFFFFF;
            font-size: 14px;
        }

        #facarticulo {
            width: 100%;
            border-collapse: collapse;
            border-spacing: 0;
            padding: 20px;
            margin-bottom: 15px;
        }

        #facarticulo thead {
            padding: 20px;
            background: #2183E3;
            text-align: center;
            border-bottom: 1px solid #FFFFFF;
        }

        #gracias {
            text-align: center;
        }
</style>
