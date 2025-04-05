<template>
  <q-page class="q-pa-md bg-grey-1">
    <div class="row q-mb-lg items-center">
      <div class="col-12 col-md-4">
        <h5 class="q-my-none text-weight-bold" style="color: #266563">
          Configurações/Campos Personalizados
        </h5>
      </div>
      <div class="col-12 col-md-8">
        <div class="row items-center q-col-gutter-md">
          <div class="col-12 col-md-4">
            <q-select
              v-model="selectedType"
              :options="typeOptions"
              label="Filtrar por Tipo"
              outlined
              dense
              clearable
              color="primary"
              class="filter-select"
              @update:model-value="applyFilters"
            >
              <template v-slot:prepend>
                <q-icon name="filter_list" color="#266563" />
              </template>
            </q-select>
          </div>
          <div class="col-12 col-md-4">
            <q-input
              v-model="search"
              placeholder="Pesquisar por Nome ou Código..."
              clearable
              outlined
              dense
              color="primary"
              class="search-input"
              @update:model-value="applyFilters"
            >
              <template v-slot:append>
                <q-icon name="search" color="#266563" />
              </template>
            </q-input>
          </div>
          <div class="col-12 col-md-4">
            <q-btn
              color="primary"
              label="Adicionar Campo"
              icon="add"
              class="full-width q-px-md"
              unelevated
              @click="showModal = true"
            />
          </div>
        </div>
      </div>
    </div>

    <q-dialog v-model="showModal" persistent>
      <q-card style="min-width: 400px; border-radius: 12px">
        <q-card-section class="bg-primary text-white">
          <div class="text-h6">Adicionar Novo Campo</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          <div class="q-mt-md">
            <q-input
              v-model="newField.name"
              label="Nome"
              outlined
              dense
              class="custom-input"
              :rules="[(val) => !!val || 'O campo Nome é obrigatório']"
            />
          </div>
          <div class="q-mt-md">
            <q-select
              v-model="newField.type"
              :options="typeOptions"
              label="Tipo"
              outlined
              dense
              class="custom-select"
              :rules="[(val) => !!val || 'O campo Tipo é obrigatório']"
            />
          </div>
          <div class="q-mt-md">
            <q-select
              v-model="newField.viewPermission"
              :options="permissionOptions"
              label="Permissão para Visualizar"
              outlined
              dense
              class="custom-select"
              :rules="[(val) => !!val || 'A permissão para visualizar é obrigatória']"
            />
          </div>
          <div class="q-mt-md">
            <q-select
              v-model="newField.editPermission"
              :options="permissionOptions"
              label="Permissão para Editar"
              outlined
              dense
              class="custom-select"
              :rules="[(val) => !!val || 'A permissão para editar é obrigatória']"
            />
          </div>
          <div class="q-mt-md">
            <q-input
              v-model="newField.order"
              label="Ordem"
              outlined
              dense
              type="number"
              class="custom-input"
              :rules="[(val) => !!val || 'A ordem é obrigatória']"
            />
          </div>
        </q-card-section>
        <q-card-actions align="right" class="q-pa-md">
          <q-btn flat label="Cancelar" color="grey-8" v-close-popup @click="resetNewField" />
          <q-btn
            color="primary"
            label="Salvar"
            unelevated
            @click="saveNewField"
            :disable="!isFormValid"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-list v-if="filteredSections.length > 0" class="column q-gutter-xl">
      <transition-group name="list" tag="div">
        <q-card
          v-for="(section, index) in filteredSections"
          :key="section.code"
          class="custom-card q-pa-md relative-position"
          style="border-radius: 16px; box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05)"
        >
          <q-btn
            round
            flat
            icon="delete"
            class="absolute delete-btn"
            style="bottom: 12px; right: 12px"
            @click="confirmDeleteSection(section, index)"
          >
            <q-tooltip>Excluir Campo</q-tooltip>
          </q-btn>

          <div class="row items-center q-col-gutter-md">
            <div class="col-12 col-md-4">
              <q-input
                v-model="section.name"
                :label="section.label"
                outlined
                dense
                class="custom-input"
                @blur="saveSection(section)"
              />
            </div>
            <div class="col-12 col-md-2">
              <q-select
                v-model="section.type"
                :options="typeOptions"
                label="Tipo"
                outlined
                dense
                class="custom-select"
                @update:model-value="saveSection(section)"
              />
            </div>
            <div class="col-12 col-md-3 flex items-center">
              <div
                class="text-subtitle2 q-mr-sm"
                :style="{ color: section.isStep ? '#266563' : '#888' }"
              >
                Etapa
              </div>
              <q-toggle
                v-model="section.isStep"
                color="primary"
                keep-color
                @update:model-value="toggleStepStatus(section)"
              />
              <div
                class="text-subtitle2 q-ml-sm q-mr-sm"
                :style="{ color: !section.isStep ? '#266563' : '#888' }"
              >
                Status
              </div>
              <q-select
                v-if="section.isStep"
                v-model="section.stepField"
                :options="stepOptions"
                :label="section.isStep ? 'Etapa' : 'Status'"
                outlined
                dense
                class="custom-select"
                style="width: 292px"
                @update:model-value="saveSection(section)"
              />
              <q-select
                v-if="!section.isStep"
                v-model="section.statusField"
                :options="statusOptions"
                :label="section.isStep ? 'Etapa' : 'Status'"
                outlined
                dense
                class="custom-select"
                style="width: 292px"
                @update:model-value="saveSection(section)"
              />
            </div>
            <div class="col-12 col-md-2">
              <q-input
                v-model="section.code"
                label="Código"
                outlined
                dense
                class="custom-input"
                @blur="saveSection(section)"
              />
            </div>
            <div class="col-12 col-md-1">
              <q-input
                v-model="section.order"
                label="Ordem"
                outlined
                dense
                type="number"
                class="custom-input"
                @blur="saveSection(section)"
              />
            </div>
          </div>

          <div class="row q-mt-md q-col-gutter-md">
            <div class="col-12 col-md-4">
              <q-select
                v-model="section.viewPermission"
                :options="permissionOptions"
                label="Permissão para Visualizar"
                outlined
                dense
                class="custom-select"
                @update:model-value="saveSection(section)"
              />
            </div>
            <div class="col-12 col-md-4">
              <q-select
                v-model="section.editPermission"
                :options="permissionOptions"
                label="Permissão para Editar"
                outlined
                dense
                class="custom-select"
                @update:model-value="saveSection(section)"
              />
            </div>
          </div>

          <div class="row q-mt-md">
            <div class="col-12">
              <q-chip
                v-for="(option, optIndex) in section.options"
                :key="optIndex"
                :label="option"
                clickable
                outlined
                color="primary"
                text-color="white"
                class="q-mr-sm q-mb-sm custom-chip"
              >
                <q-btn
                  round
                  flat
                  dense
                  icon="close"
                  color="white"
                  size="sm"
                  @click="confirmDeleteOption(section, optIndex)"
                >
                  <q-tooltip>Excluir Opção</q-tooltip>
                </q-btn>
              </q-chip>
              <q-btn
                color="primary"
                label="Adicionar Opção"
                flat
                icon="add"
                class="add-option-btn"
                text-color="white"
                @click="showOptionModal(index)"
              />
            </div>
          </div>
        </q-card>
      </transition-group>
    </q-list>
    <div v-else class="text-center q-mt-lg">
      <q-icon name="info" size="lg" color="grey-6" />
      <p class="text-grey-6 q-mt-sm">Nenhum campo encontrado para os filtros aplicados.</p>
    </div>

    <q-dialog v-model="showOptionModalFlag" persistent>
      <q-card style="min-width: 300px; border-radius: 12px">
        <q-card-section class="bg-primary text-white">
          <div class="text-h6">Adicionar Opção</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          <div class="q-mt-md">
            <q-input
              v-model="newOption"
              label="Nome da Opção"
              outlined
              dense
              class="custom-input"
              :rules="[(val) => !!val || 'O nome da opção é obrigatório']"
            />
          </div>
        </q-card-section>
        <q-card-actions align="right" class="q-pa-md">
          <q-btn flat label="Cancelar" color="grey-8" v-close-popup @click="resetNewOption" />
          <q-btn
            color="primary"
            label="Adicionar"
            unelevated
            @click="addOption"
            :disable="!newOption"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Modal de Confirmação para Deletar Campo -->
    <q-dialog v-model="showDeleteSectionModal" persistent>
      <q-card style="min-width: 400px; border-radius: 12px">
        <q-card-section class="bg-red-9 text-white">
          <div class="text-h6">Confirmar Exclusão de Campo</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          <p class="q-mt-md">
            Você está prestes a excluir o campo "<strong>{{ sectionToDelete?.name }}</strong
            >". Esta ação é <strong>permanente</strong> e não pode ser desfeita. Deseja continuar?
          </p>
        </q-card-section>
        <q-card-actions align="right" class="q-pa-md">
          <q-btn flat label="Cancelar" color="grey-8" v-close-popup />
          <q-btn
            color="red-9"
            label="Excluir Permanentemente"
            unelevated
            @click="deleteSectionConfirmed"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Modal de Confirmação para Deletar Opção -->
    <q-dialog v-model="showDeleteOptionModal" persistent>
      <q-card style="min-width: 400px; border-radius: 12px">
        <q-card-section class="bg-red-9 text-white">
          <div class="text-h6">Confirmar Exclusão de Opção</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          <p class="q-mt-md">
            Você está prestes a excluir a opção "<strong>{{ optionToDelete?.label }}</strong
            >". Esta ação é <strong>permanente</strong> e não pode ser desfeita. Deseja continuar?
          </p>
        </q-card-section>
        <q-card-actions align="right" class="q-pa-md">
          <q-btn flat label="Cancelar" color="grey-8" v-close-popup />
          <q-btn
            color="red-9"
            label="Excluir Permanentemente"
            unelevated
            @click="deleteOptionConfirmed"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
export default {
  data() {
    return {
      search: '',
      selectedType: null,
      showModal: false,
      showOptionModalFlag: false,
      showDeleteSectionModal: false,
      showDeleteOptionModal: false,
      currentSectionIndex: null,
      newOption: '',
      sectionToDelete: null,
      sectionIndexToDelete: null,
      optionToDelete: null,
      optionIndexToDelete: null,
      typeOptions: [
        'Arquivo',
        'Assinatura',
        'Data e Hora',
        'Localização',
        'Rádio',
        'Texto Longo',
        'Arquivos',
        'Check List',
        'Data',
        'Hora',
        'Número',
        'Texto Curto',
        'Fórmula',
      ],
      permissionOptions: ['Administrador', 'Supervisor', 'Gerente', 'Vendedor'],
      stepOptions: ['PROSPECÇÃO', 'VENDAS'],
      statusOptions: ['STATUS A', 'STATUS B'],
      newField: {
        name: '',
        type: null,
        viewPermission: null,
        editPermission: null,
        order: '',
      },
      sections: [
        {
          name: 'Grécia',
          label: 'Nome',
          type: 'Texto Longo',
          isStep: true,
          stepField: '',
          statusField: '',
          code: 'greece',
          order: '1',
          viewPermission: 'Administrador',
          editPermission: 'Administrador',
          options: [],
        },
        {
          name: 'Albânia',
          label: 'Nome',
          type: 'Check List',
          isStep: true,
          stepField: '',
          statusField: '',
          code: 'albania',
          order: '2',
          viewPermission: 'Administrador',
          editPermission: 'Administrador',
          options: ['opção 1', 'opção 2', 'opção 3'],
        },
        {
          name: 'Kosovo',
          label: 'Nome',
          type: 'Rádio',
          isStep: true,
          stepField: '',
          statusField: '',
          code: 'witheggs',
          order: '3',
          viewPermission: 'Administrador',
          editPermission: 'Administrador',
          options: ['Pequeno', 'Novo', 'Fácil de achar'],
        },
      ],
      filteredSections: [],
    }
  },
  computed: {
    isFormValid() {
      return (
        !!this.newField.name &&
        !!this.newField.type &&
        !!this.newField.viewPermission &&
        !!this.newField.editPermission &&
        !!this.newField.order
      )
    },
  },
  methods: {
    applyFilters() {
      let filtered = [...this.sections]
      if (this.selectedType) {
        filtered = filtered.filter((section) => section.type === this.selectedType)
      }
      if (this.search) {
        const searchLower = this.search.toLowerCase()
        filtered = filtered.filter(
          (section) =>
            section.name.toLowerCase().includes(searchLower) ||
            section.code.toLowerCase().includes(searchLower),
        )
      }
      filtered.sort((a, b) => {
        const orderA = parseInt(a.order, 10) || 0
        const orderB = parseInt(b.order, 10) || 0
        return orderA - orderB
      })
      this.filteredSections = filtered
    },
    saveSection(section) {
      const originalIndex = this.sections.findIndex((s) => s.code === section.code)
      if (originalIndex !== -1) {
        section.order = parseInt(section.order, 10).toString()
        this.sections[originalIndex] = { ...section }
        this.applyFilters()
      }
    },
    toggleStepStatus(section) {
      section.stepField = ''
      section.statusField = ''
      this.saveSection(section)
    },
    confirmDeleteSection(section, index) {
      this.sectionToDelete = section
      this.sectionIndexToDelete = index
      this.showDeleteSectionModal = true
    },
    deleteSectionConfirmed() {
      const originalIndex = this.sections.findIndex((s) => s.code === this.sectionToDelete.code)
      if (originalIndex !== -1) {
        this.sections.splice(originalIndex, 1)
        this.filteredSections.splice(this.sectionIndexToDelete, 1)
        this.applyFilters()
      }
      this.showDeleteSectionModal = false
      this.sectionToDelete = null
      this.sectionIndexToDelete = null
    },
    confirmDeleteOption(section, optionIndex) {
      this.optionToDelete = { label: section.options[optionIndex], section }
      this.optionIndexToDelete = optionIndex
      this.showDeleteOptionModal = true
    },
    deleteOptionConfirmed() {
      const originalSection = this.sections.find((s) => s.code === this.optionToDelete.section.code)
      if (originalSection) {
        originalSection.options.splice(this.optionIndexToDelete, 1)
        this.applyFilters()
      }
      this.showDeleteOptionModal = false
      this.optionToDelete = null
      this.optionIndexToDelete = null
    },
    showOptionModal(sectionIndex) {
      this.currentSectionIndex = sectionIndex
      this.showOptionModalFlag = true
    },
    addOption() {
      if (!this.newOption) return
      const section = this.filteredSections[this.currentSectionIndex]
      const originalIndex = this.sections.findIndex((s) => s.code === section.code)
      if (originalIndex !== -1) {
        this.sections[originalIndex].options.push(this.newOption)
        this.applyFilters()
        this.showOptionModalFlag = false
        this.resetNewOption()
      }
    },
    resetNewOption() {
      this.newOption = ''
      this.currentSectionIndex = null
    },
    resetNewField() {
      this.newField = {
        name: '',
        type: null,
        viewPermission: null,
        editPermission: null,
        order: '',
      }
    },
    saveNewField() {
      if (!this.isFormValid) return
      const newSection = {
        name: this.newField.name,
        label: 'Nome',
        type: this.newField.type,
        isStep: false,
        stepField: '',
        statusField: '',
        code: this.newField.name.toLowerCase().replace(/\s+/g, '_'),
        order: parseInt(this.newField.order, 10).toString(),
        viewPermission: this.newField.viewPermission,
        editPermission: this.newField.editPermission,
        options: [],
      }
      this.sections.push(newSection)
      this.applyFilters()
      this.showModal = false
      this.resetNewField()
    },
  },
  created() {
    this.filteredSections = [...this.sections]
    this.applyFilters()
  },
}
</script>

<style scoped>
.q-page {
  background: linear-gradient(135deg, #f5f7fa 0%, #e4e7eb 100%);
}
h5 {
  font-size: 1.5rem;
  color: #266563;
}
.custom-card {
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  background: #fff;
  transition: transform 0.2s ease-in-out;
  margin: 16px 0;
}
.custom-card:hover {
  transform: translateY(-4px);
}
.custom-input,
.custom-select {
  background-color: #fff;
  border-radius: 8px;
}
.custom-input .q-field__control,
.custom-select .q-field__control {
  border: 1px solid #e0e0e0;
}
.filter-select,
.search-input {
  background-color: #fff;
  border-radius: 8px;
}
.filter-select .q-field__control,
.search-input .q-field__control {
  border: 1px solid #266563;
}
.custom-chip {
  border-radius: 16px;
  transition: background-color 0.3s;
}
.custom-chip:hover {
  background-color: #d4e6e5;
}
.custom-btn {
  border-radius: 8px;
  padding: 8px 16px;
  font-weight: 500;
  transition: all 0.3s ease;
}
.custom-btn:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  transform: translateY(-2px);
}
.q-btn[color='primary'] {
  background: linear-gradient(90deg, #266563 0%, #3b8a88 100%);
  color: #fff !important;
}
.q-btn[color='primary'] .q-icon {
  color: #fff !important;
}
.q-btn[color='white'] {
  color: #fff !important;
}
.add-option-btn {
  color: #266563 !important;
}
.add-option-btn .q-icon {
  color: #266563 !important;
}
.delete-btn {
  background: linear-gradient(90deg, #266563 0%, #3b8a88 100%);
  color: #fff !important;
}
.delete-btn .q-icon {
  color: #fff !important;
}
.bg-primary {
  background: #266563 !important;
}
.q-icon {
  color: #266563 !important;
}
.q-toggle {
  color: #266563 !important;
}
.q-toggle .q-toggle__inner {
  color: #266563 !important;
}
.q-toggle .q-toggle__inner--truthy {
  color: #266563 !important;
}
@media (max-width: 600px) {
  h5 {
    font-size: 1.2rem;
  }
  .custom-btn {
    padding: 6px 12px;
  }
}

.list-move,
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
.list-leave-active {
  position: absolute;
}
</style>
