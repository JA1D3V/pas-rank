<template>
  <div id="app" class='container-float'>
    <h1>PAS Ranking</h1>
    <p>Site para facilitar o entendimento da sua classificação no PAS</p>
    <div class='action'>
      <div class='row'>
        <div class='col content'>
          <b-form-group id="selCursos"
                    label="Selecione seu curso:"
                    label-for="cursos">
            <b-form-select id="cursos"
                          :options="cursos"
                          required
                          v-model="curso"
                          v-on:input="fetchData">
            </b-form-select>
          </b-form-group>
          <b-form-group v-if='sizeRanking' id='rankingGroup' label='Insira sua classificação geral (sem "°")' label-for='ranking'>
            <p>De 1° a {{sizeRanking}}°</p>
            <b-form-input type='number' v-model='ranking' min='1' :max='sizeRanking'></b-form-input>
          </b-form-group>
          <b-form-group id="selCotas"
                    label="Selecione a concorrência:"
                    label-for="conc">
            <b-form-select id="conc"
                          :options="concorrencias"
                          required
                          v-model="conc"
                          v-on:input="calcResult">
            </b-form-select>
            <div class='btn-holder'>
              <b-button v-b-modal.modal1 class='btn' variant="primary">
                Legendas
              </b-button>
            </div>
          </b-form-group>
        </div>
        <div v-if='concF'>
          <h3><b>Resultado:</b></h3>
          <h4>Você possui {{concG.length}} concorrentes no geral e está posicionado em {{concF.length}}°</h4>
          <h4 v-if='actualp != 0'>Atualmente você é o {{actualp}}° candidato na ordem de chamada</h4>  
          <h4 v-if='actualp == 0'>Você provavelmente já foi convocado 🤔</h4>
        </div>
      </div>
    </div>
    <b-modal id="modal1" title="Legenda para concorrência" class='mdl'>
      <p class="my-4">(1) Sistema Universal <br><br>
        (2) Sistema de Cotas para Negros <br><br>
        (3) Sistema de Cotas para Escolas Públicas/Candidatos com renda familiar bruta igual ou inferior a 1,5
        salário mínimo per capita que se autodeclararam pretos, pardos ou indígenas <br><br>
        (4) Sistema de Cotas para Escolas Públicas/Candidatos com renda familiar bruta igual ou inferior a 1,5
        salário mínimo per capita que se autodeclararam pretos, pardos ou indígenas e que concorrem como
        pessoas com deficiência <br><br>
        (5) Sistema de Cotas para Escolas Públicas/Candidatos com renda familiar bruta igual ou inferior a 1,5
        salário mínimo per capita que não se autodeclararam pretos, pardos ou indígenas <br><br>
        (6) Sistema de Cotas para Escolas Públicas/Candidatos com renda familiar bruta igual ou inferior a 1,5
        salário mínimo per capita que não se autodeclararam pretos, pardos ou indígenas e que concorrem como
        pessoas com deficiência <br><br>
        (7) Sistema de Cotas para Escolas Públicas/Candidatos com renda familiar bruta superior a 1,5 salário
        mínimo per capita que se autodeclararam pretos, pardos ou indígenas <br><br>
        (8) Sistema de Cotas para Escolas Públicas/candidatos com renda familiar bruta superior a 1,5 salário
        mínimo per capita que se autodeclararam pretos, pardos ou indígenas e que concorrem como pessoas
        com deficiência <br><br>
        (9) Sistema de Cotas para Escolas Públicas/candidatos com renda familiar bruta superior a 1,5 salário
        mínimo per capita que não se autodeclararam pretos, pardos ou indígenas <br><br>
        (10) Sistema de Cotas para Escolas Públicas/candidatos com renda familiar bruta superior a 1,5 salário
        mínimo per capita que não se autodeclararam pretos, pardos ou indígenas e que concorrem como pessoas
        com deficiência
      </p>
    </b-modal>
    <b-modal ref="errmodal" class='err-modal' title="Revise as informações!" @shown="clear()">
      <h2 class="my-4">O seu resultado foi calculado, porém algo esta errado!</h2>
      <h3>Como corrigir?</h3>
      <p><b>1) </b> Confirme sua classificação geral, ela está presente no espelho de desempenho disponibilizado no site do CESPE, <a href='http://bit.ly/espelhodes'>Clique aqui para ir até lá</a></p>
      <img src='./assets/h1.png' class='help-img' alt='imagem de referencia' />
      <p><b>2) </b> Confirme o seu sistema de concorrência, caso não saiba, clique no botão "legendas" para ver o detalhamento de cada tipo.</p>

    </b-modal>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'app',
  data () {
    return {
      cursos: null,
      concorrencias: [
        {value: '1', text: '1'},
        {value: '2', text: '2'},
        {value: '3', text: '3'},
        {value: '4', text: '4'},
        {value: '5', text: '5'},
        {value: '6', text: '6'},
        {value: '7', text: '7'},
        {value: '8', text: '8'},
        {value: '9', text: '9'},
        {value: '10', text: '10'},
      ],
      curso: null,
      conc: null,
      sizeRanking: null,
      ranking: null,
      overall: [],
      decompiled: null,
      already: null,
      compare: function(a,b) {
        if (Number(a[12]) < Number(b[12]))
          return -1;
        if (Number(a[12]) > Number(b[12]))
          return 1;
        return 0;
      },
      concG: null,
      concF: null,
      actualp: null
    }
  },
  methods: {
    clear () {
      this.ranking = null
      this.conc = null
      this.concG = null
      this.concF = null
    },
    fetchData () {
      if (this.curso) {
        axios.get('listas/'+this.curso).then(x => {
          console.log('received')
          this.treatment(x.data)
        })
      }
    },
    treatment (data) {
      let v1 = data.split('=')
      this.already = v1[0]
      let decompiled = v1[1].split('/')
      this.sizeRanking = decompiled.length
      this.decompiled = decompiled
      this.makeMagic(decompiled)
    },
    
    makeMagic (data) {
      let tmp = []
      for(let b = 0; b < data.length; b++){
        tmp.push(data[b].split(','))
      }
      this.overall = tmp.sort(this.compare)
    },
    calcResult () {
      let concorrentesGeral = [];
      let concorrentesFrente = []
      for (let a = 0; a < this.overall.length; a++) {
        let act = Number(this.conc) + 11
        if(act != 12){
          if(this.overall[a][act] == '-'){
            console.log('jmp')
          }else{
            concorrentesGeral.push(this.overall[a])
            if (Number(this.overall[a][12]) < Number(this.ranking)) {
              concorrentesFrente.push(this.overall[a])
            }
          }
        }else{
          if(this.overall[a][13] == '-'
            && this.overall[a][14] == '-'
            && this.overall[a][15] == '-'
            && this.overall[a][16] == '-'
            && this.overall[a][17] == '-'
            && this.overall[a][18] == '-'
            && this.overall[a][19] == '-'
            && this.overall[a][20] == '-'
            && this.overall[a][21] == '-'){
              concorrentesGeral.push(this.overall[a])
              if (Number(this.overall[a][12]) <= Number(this.ranking)) {
                if(Number(this.overall[a][12]) > 10){
                  console.log(this.overall[a][12] + '° ' + this.overall[a][1])
                }
                concorrentesFrente.push(this.overall[a])
              }
          }
        }
      }
      this.concG = concorrentesGeral
      if(concorrentesFrente < 1){
        this.$refs.errmodal.show()
      } else {
        let sub = this.already.split(',')
        let spec = sub[Number(this.conc) - 1]
        let xls = concorrentesFrente.length - Number(spec)
        if(xls > 0){
          this.actualp = xls
        } else {
          this.actualp = 0
        }
        this.concF = concorrentesFrente
      }
    }
  },
  beforeMount () {
    axios.get('listas/dynamic').then(x => {
      console.log(x.data)
      this.cursos = x.data
    })
  }
}
</script>

<style>
#app {
  height: 100vh;
  width: 100vw;
  background-color: #094074;
  text-align: center;
  color: white;
}
.action {
  background-color: white;
  color: black;
  height: fit-content;
  width: 75vw;
  margin: auto;
  border-radius: 50px;
  padding: 5%;
}

.content {
  margin: auto;
}

.btn {
  margin-top: 1%;
}

.btn-holder {
  text-align: right;
}

.mdl {
  color: black
}

.help-img {
  max-width: 70%;
}

.err-modal {
  color: black;
}
</style>
