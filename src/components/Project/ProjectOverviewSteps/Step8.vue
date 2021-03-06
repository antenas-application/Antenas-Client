<template>
  <div v-if="isTeacher" class="project-step-8">
    <el-alert
      :closable="false"
      title="Avalie e atribua medalhas aos alunos"
      type="info"
    >
      <el-button
        :loading="loading"
        type="primary"
        class="ml-16"
        icon="el-icon-notebook-2"
        @click="openDialog()"
      >
        Avaliar alunos
      </el-button>
    </el-alert>

    <el-dialog
      title="Avaliação"
      width="80%"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
    >
      <el-steps
        :active="step"
        finish-status="success"
        process-status="finish"
        simple
      >
        <el-step title="Atribuir medalhas" icon="el-icon-medal-1" />
        <el-step title="Atribuir notas" icon="el-icon-notebook-2" />
      </el-steps>
      <div v-if="!step">
        <el-row :gutter="32">
          <el-col :span="10" class="medals">
            <div class="d-flex">
              <el-input
                v-model="searchTerm"
                placeholder="Buscar medalhas"
                suffix-icon="el-icon-search"
                class="w100 mr-12"
                clearable
              />
              <el-button type="primary" @click="newMedalVisible = true">
                Criar nova medalha
              </el-button>
            </div>
            <div class="medals-list">
              <el-row>
                <div
                  v-for="medal in filteredMedals"
                  :key="medal.id"
                  draggable="true"
                  class="drag"
                  @dragstart="dragStart($event, medal.id)"
                >
                  <el-col :span="8" class="text-center mt-16">
                    <medal-template :medal="medal" style="pointer-events: none;" />
                  </el-col>
                </div>
              </el-row>
            </div>
          </el-col>
          <el-col :span="14" class="teams">
            <div
              v-for="team in teams"
              :id="`team-${team.id}`"
              :key="team.id"
              class="team"
              @dragover.prevent
              @dragenter="onDragEnter($event, `team-${team.id}`)"
              @dragleave="onDragLeave($event, `team-${team.id}`)"
              @drop="onDropTeam($event, team, `team-${team.id}`)"
            >
              <h1>{{ team.name }}</h1>
              <div
                v-for="student in team.studentTeamList"
                :id="`student-${student.id}`"
                :key="student.id"
                class="student"
                @dragover.prevent
                @dragenter="onDragEnter($event, `student-${student.id}`)"
                @dragleave="onDragLeave($event, `student-${student.id}`)"
                @drop="onDropStudent($event, student, `student-${student.id}`)"
              >
                <div class="d-flex align-center">
                  <span class="student-name">
                    {{ student.student.name }}
                  </span>
                  <div
                    v-for="studentMedal in student.student.studentMedals"
                    :key="studentMedal.medal.name"
                    class="medal-student"
                    @click="removeMedal()"
                  >
                    <medal-template v-if="!studentMedal.date" :medal="studentMedal.medal" />
                  </div>
                </div>
              </div>
            </div>
          </el-col>
        </el-row>
      </div>
      <div v-else class="evaluation">
        <div v-for="team in teams" :key="team.id" class="team-info">
          <span class="team-name"> {{ team.name }} </span>
          <el-row v-for="studentTeam in team.studentTeamList" :key="studentTeam.id" class="student-evaluation">
            <el-col :span="24">
              <el-row :gutter="24">
                <el-col :span="5" class="student-name">
                  {{ studentTeam.student.name }}
                </el-col>
                <el-col :span="4">
                  Proatividade
                  <el-rate
                    v-model="studentTeam.evaluation.proactivity"
                    :colors="colors"
                  />
                </el-col>
                <el-col :span="4">
                  Autonomia
                  <el-rate
                    v-model="studentTeam.evaluation.autonomy"
                    :colors="colors"
                  />
                </el-col>
                <el-col :span="4">
                  Colaboração
                  <el-rate
                    v-model="studentTeam.evaluation.collaboration"
                    :colors="colors"
                  />
                </el-col>
                <el-col :span="4">
                  Entrega de resultados
                  <el-rate
                    v-model="studentTeam.evaluation.resultsDeliver"
                    :colors="colors"
                  />
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </div>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">Cancelar</el-button>
        <el-button v-if="step === 1" icon="el-icon-back" circle @click="step = 0" />
        <el-button v-if="step === 0" icon="el-icon-right" circle @click="step = 1" />
        <el-button v-if="step === 1" type="primary" @click="evaluate()">Salvar Avaliação</el-button>
      </span>
    </el-dialog>
    <MedalNew :visible.sync="newMedalVisible" />
  </div>
</template>

<script>
import MedalTemplate from '@/components/Medal/MedalTemplate'
import MedalNew from '@/components/Medal/MedalNew'

import TeamService from '@/services/TeamService'

import { mapGetters } from 'vuex'

export default {
  components: {
    MedalTemplate,
    MedalNew
  },
  data () {
    return {
      newMedalVisible: false,
      dialogVisible: false,
      step: 0,
      loading: false,
      searchTerm: '',
      teams: [],
      value: null,
      colors: ['#99A9BF', '#F7BA2A', '#FF9900']
    }
  },
  computed: {
    ...mapGetters([
      'selectedProject',
      'isTeacher',
      'medals',
      'getMedalById'
    ]),
    filteredMedals () {
      return this.searchTerm ? this.medals.filter(m => m.name.includes(this.searchTerm)) : this.medals
    }
  },
  methods: {
    openDialog () {
      this.loading = true
      this.getTeams()
        .then(response => {
          this.teams = response
          this.dialogVisible = true
        })
        .catch(err => this.$throwError(err))
        .finally(() => { this.loading = false })
    },
    getTeams () {
      return TeamService.getTeam(this.selectedProject.id).then(res => res)
    },
    searchMedals () {
    },
    dragStart (ev, medalId) {
      ev.dataTransfer.setData('medalId', medalId)
      ev.dataTransfer.dropEffect = 'move'
      ev.dataTransfer.effectAllowed = 'move'
    },
    onDropTeam (ev, team, elIdd) {
      ev.stopPropagation()
      document.getElementById(elIdd).classList.remove('is-dragging')
      const medal = this.getMedalById(ev.dataTransfer.getData('medalId'))
      team.studentTeamList.forEach(s => {
        const medalExists = s.student.studentMedals.find(m => m.id === medal.id)
        if (!medalExists) s.student.studentMedals.push({ medal: medal })
      })
    },
    onDropStudent (ev, student, elIdd) {
      ev.stopPropagation()
      document.getElementById(elIdd).classList.remove('is-dragging')
      const medal = this.getMedalById(ev.dataTransfer.getData('medalId'))
      const medalExists = student.student.studentMedals.find(m => m.id === medal.id)
      if (!medalExists) student.student.studentMedals.push({ medal: medal })
    },
    onDragEnter (ev, id) {
      ev.stopPropagation()
      document.getElementById(id).classList.add('is-dragging')
    },
    onDragLeave (ev, id) {
      ev.stopPropagation()
      document.getElementById(id).classList.remove('is-dragging')
    },
    evaluate () {
      this.dialogVisible = false
      this.$store.commit('SHOW_LOADING')
      this.teams.forEach(team => {
        team.project.id = this.selectedProject.id
      })
      TeamService.evaluate(this.teams)
        .catch(err => this.$throwError(err))
        .finally(() => this.$store.commit('HIDE_LOADING'))

      // this.$store.dispatch('updateProjectView', this.project)
      //   .catch(err => this.$throwError(err))
      //   .finally(() => this.$store.commit('HIDE_LOADING'))
    }
  }
}
</script>

<style lang="scss">
@import '@/styles/_colors.scss';

.project-step-8 {
  .el-dialog {
    min-height: 50vh;
  }
  .medals, .teams {
    padding: 32px 0 0;
    h1 {
      font-size: 1.17rem;
      pointer-events: none;
    }
    .student-name {
      font-size: 1rem;
      padding: 9px 0;
      pointer-events: none;
    }
  }
  .medals-list {
    padding-top: 16px;
  }
  .medal-template + .medal-template {
    margin-top: 12px;
  }
  .team {
    padding: 12px;
    border: 1px dashed;
    border-left: 5px solid;
    border-radius: 4px;
    background-color: white;
    &.is-dragging {
      background-color: #F5F7FA;
      border-color: #4472E9;
    }
  }
  .team + .team {
    margin-top: 20px;
  }
  .student {
    border: 1px dashed;
    padding: 8px 12px;
    margin-top: 8px;
    background-color: white;
    &.is-dragging {
      border-color: #4472E9;
      background-color: #F5F7FA;
    }
    > div {
      pointer-events: none;
    }
  }
  .drag {
    opacity: 0.999;
  }
  .medal-student {
    transform: scale(0.2);
    width: 40px;
    height: 34px;
    display:inline-block;
    transform-origin: top center;
    cursor: pointer;
    &:hover {
      &::before {
        content: "\e7c9";
        font-family: element-icons !important;
        font-size: 90px;
        position: absolute;
        z-index: 9999;
        color: #DD2C2C;
        background-color: #ffffff;
        padding: 40px 24px;
      }
    }
  }

  .student-name {
    line-height: 30px;
    font-weight: 500;
  }
  .team-name {
    font-weight: 600;
    font-size: 20px;
  }
  .student-evaluation {
    border: solid #c3c8d68f 1px;
    margin: 10px 0px;
    padding: 10px;
    border-radius: 10px;
  }
  .team-info {
    border-bottom: solid #c3c8d6 2px;
    margin-bottom: 14px;
    padding-bottom: 8px;
  }
  .evaluation {
    height: 516px;
    max-height: 100%;
    overflow-y: auto;
    margin-top: 20px;
    overflow-x: hidden;
  }
}
</style>
