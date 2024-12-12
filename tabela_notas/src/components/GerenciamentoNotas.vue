<!-- SCRIPT -->
 <script>
//Importação do Bootstrap
import 'bootstrap/dist/css/bootstrap.min.css';

export default {
  // O nome do componente, usado para identificar o componente no DevTools ou ao registrá-lo em outros lugares.
  name: 'GerencimentoNotas',

  // Aqui defino o "data", que retorna um objeto com as variáveis que controlam o estado do componente.
  data() {
    return {
      alunos: [], // Lista de alunos cadastrados
      novoAluno: { nome: '', media1: '', media2: '', situacao: '' }, // Objeto usado para adicionar ou editar alunos
      editando: false, // Flag para saber se estamos editando ou criando um novo aluno
      indexEdicao: null, // Armazena o índice do aluno que está sendo editado
      termoPesquisa: '', // Variável para armazenar o termo de pesquisa
      formSubmetido: false, // Para controlar se o formulário foi enviado
    };
  },

  // O `methods` contém as funções/métodos que realizam ações no componente, como adicionar, editar ou remover alunos.
  methods: {
    validarNota(campo) {
      let valor = this.novoAluno[campo];

      // Permitir apenas números, vírgulas e pontos
      valor = valor.replace(/[^0-9.,]/g, '');

      // Substituir múltiplos pontos ou vírgulas pelo último digitado
      const partes = valor.split(/[.,]/);
      if (partes.length > 2) {
        valor = partes[0] + ',' + partes[1]; // Permitir apenas uma separação decimal
      }

      // Substituir ponto por vírgula para exibição
      valor = valor.replace('.', ',');

      // Verificar se o número está no intervalo de 0 a 10
      const numero = parseFloat(valor.replace(',', '.'));
      if (numero < 0 || numero > 10) {
        valor = ''; // Limpar o campo se fora do intervalo
      }

      // Atualizar o campo com o valor validado
      this.novoAluno[campo] = valor;
    },


    // Método para calcular a situação do aluno com base nas notas
    calcularSituacao(media1, media2) {
      const nota1 = parseFloat(media1.replace(',', '.'));
      const nota2 = parseFloat(media2.replace(',', '.'));

      // Calcular a média
      const media = (nota1 + nota2) / 2;

      // Determinar a situação
      let situacao = '';
      if (media >= 7) {
        situacao = 'Aprovado';
      } else if (media >= 5) {
        situacao = 'Em Exame';
      } else {
        situacao = 'Reprovado';
      }

      return {
        media: media.toFixed(2), // Arredonda para duas casas decimais
        situacao: situacao
      };
    },

    validarCampos() {
      // Verifica se algum campo está vazio
      if (!this.novoAluno.nome || !this.novoAluno.media1 || !this.novoAluno.media2) {
        return false; // Se algum campo estiver vazio, não permite o cadastro
      }
      return true; // Se todos os campos estiverem preenchidos
    },


    // Método para cadastrar ou atualizar alunos
    cadastrar() {
      this.formSubmetido = true; // Ativa a verificação de submissão do formulário

      // Validar se todos os campos estão preenchidos
      if (!this.validarCampos()) {
        return; // Não prossegue se algum campo estiver vazio
      }

      // Calcular a situação do aluno com as notas fornecidas
      const situacao = this.calcularSituacao(this.novoAluno.media1, this.novoAluno.media2);

      // Aqui utilizo o this para acessar propriedades e métodos do componente Vue. Ao invés do then mostrado no curso (mais indicado para operação assíncronas)
      if (this.editando) {
        // Se estamos editando, atualiza o aluno na lista
        this.alunos[this.indexEdicao] = {
          ...this.novoAluno,
          media: situacao.media,
          situacao: situacao.situacao,
        };
        this.editando = false;
        this.indexEdicao = null;
      } else {
        // Se estamos cadastrando um novo aluno
        this.alunos.push({
          ...this.novoAluno,
          media: situacao.media,
          situacao: situacao.situacao,
        });
      }
      // Limpa os campos do formulário e reseta o estado do formulário
      this.novoAluno = { nome: '', media1: '', media2: '', situacao: '' };
      this.formSubmetido = false; // Reset do estado de submissão
      this.salvarNoLocalStorage(); // Salva os dados no LocalStorage
    },

    // Método para carregar os dados de um aluno no formulário, permitindo edição
    editar(index) {
      this.novoAluno = { ...this.alunos[index] }; // Copia os dados do aluno para o formulário
      this.editando = true; // Ativa o modo de edição
      this.indexEdicao = index; // Salva o índice do aluno que está sendo editado
    },

    // Método para remover um aluno da lista
    remover(index) {
      this.alunos.splice(index, 1); // Remove o aluno pelo índice
      this.salvarNoLocalStorage(); // Atualiza o Local Storage
    },

    // Salva os dados no Local Storage
    salvarNoLocalStorage() {
      localStorage.setItem('alunos', JSON.stringify(this.alunos));
    },

    //Carrega os dados do Local Storage
    carregarDoLocalStorage() {
      const dados = localStorage.getItem('alunos');
      if (dados) {
        this.alunos = JSON.parse(dados);
      }
    },

    //Exportar lista em JSON
    exportarJSON() {
      // Converter a lista de alunos para JSON
      const dadosJSON = JSON.stringify(this.alunos, null, 2);

      // Criar um blob para o arquivo
      const blob = new Blob([dadosJSON], { type: 'application/json;charset=utf-8;' });

      // Criar um link para download
      const link = document.createElement('a');
      const url = URL.createObjectURL(blob);
      link.href = url;
      link.setAttribute('download', 'alunos.json');

      // Simular clique no link
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
  },
  mounted() {
    this.carregarDoLocalStorage();
  },
};
</script>

<!-- HMTL -->
<template>
  <body>
    <div class="grid-container">
      <h1 class="titulo">Bem-vindo ao Sistema de Gerenciamento de Notas</h1>

      <!-- Formulário para cadastrar ou editar alunos -->
      <!-- Formulário para cadastrar ou editar alunos -->
      <form class="from-group formulario" @submit.prevent="cadastrar">
        <div>
          <input
            class="form-control"
            type="text"
            v-model="novoAluno.nome"
            placeholder="Nome do Aluno"
            :class="{ 'is-invalid': formSubmetido && !novoAluno.nome }"
            required
          />
          <div v-if="formSubmetido && !novoAluno.nome" class="invalid-feedback">
            O nome do aluno é obrigatório.
          </div>
        </div>

        <div>
          <input
            class="form-control"
            type="text"
            v-model="novoAluno.media1"
            placeholder="1ª Nota"
            :class="{ 'is-invalid': formSubmetido && !novoAluno.media1 }"
            required
            @input="validarNota('media1')"
          />
          <div
            v-if="formSubmetido && !novoAluno.media1"
            class="invalid-feedback"
          >
            A 1ª nota é obrigatória.
          </div>
        </div>

        <div>
          <input
            class="form-control"
            type="text"
            v-model="novoAluno.media2"
            placeholder="2ª Nota"
            :class="{ 'is-invalid': formSubmetido && !novoAluno.media2 }"
            @input="validarNota('media2')"
          />
          <div
            v-if="formSubmetido && !novoAluno.media2"
            class="invalid-feedback"
          >
            A 2ª nota é obrigatória.
          </div>
        </div>

        <button class="btn btn-success cadastrar" type="submit">
          {{ editando ? "Salvar Alterações" : "Cadastrar" }}
        </button>
      </form>

      <div class="grid-exportar">
        <button @click="exportarJSON" class="btn btn-secondary exportar">
          Exportar Lista
        </button>
      </div>

      <!-- Tabela para exibir a lista de alunos -->
      <div v-if="alunos.length === 0" class="alert alert-info text-center">
        Nenhum aluno cadastrado.
      </div>

      <div class="grid-exportar">
        <input
          v-if="alunos.length > 0"
          type="text"
          class="form-control form-control-sm filtrar"
          placeholder="🔍︎ Pesquisar aluno"
          v-model="termoPesquisa"
        />
      </div>

      <table class="table table-hover table-dark">
        <thead>
          <tr>
            <th>Nome</th>
            <th>1ª Nota</th>
            <th>2ª Nota</th>
            <th>Média Geral</th>
            <th>Situação</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(aluno, index) in alunos.filter((aluno) =>
              aluno.nome.toLowerCase().includes(termoPesquisa.toLowerCase())
            )"
            :key="index"
          >
            <td>{{ aluno.nome }}</td>
            <td>{{ aluno.media1 }}</td>
            <td>{{ aluno.media2 }}</td>
            <td>{{ aluno.media }}</td>
            <!-- Exibe a média calculada -->
            <td
              :class="{
                aprovado: aluno.situacao === 'Aprovado',
                emExame: aluno.situacao === 'Em Exame',
                reprovado: aluno.situacao === 'Reprovado',
              }"
            >
              {{ aluno.situacao }}
              <!-- Exibe a situação calculada -->
            </td>
            <td>
              <button
                class="btn btn-sm btn-warning editar"
                @click="editar(index)"
              >
                Editar
              </button>
              <button
                class="btn btn-sm btn-danger remover"
                @click="remover(index)"
              >
                Remover
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </body>
</template>


<!-- CSS -->
<style scoped>
body {
  background: rgb(8, 16, 22);
  background: radial-gradient(
    circle,
    rgba(8, 16, 22, 1) 0%,
    rgba(66, 114, 149, 1) 100%
  );
  margin: 0 !important;
  padding: 0 !important;
}
.grid-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
}
.titulo {
  color: #fffafd;
  margin-top: 20px;
  font-family: "Roboto", sans-serif;
  font-weight: 400;
  font-style: normal;
  font-size: 3rem;
  text-align: center;
  margin-bottom: 20px;
}
.formulario {
  display: grid;
  place-items: center;
  text-align: center;
}
.formulario input {
  font-family: "Roboto", sans-serif;
  font-style: normal;
  font-size: 1.1rem;
  margin-bottom: 4px;
}
table {
  width: 80%;
  margin: 15px auto;
  border-collapse: separate;
  border-spacing: 0;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 0;
}
tbody {
  display: table-row-group;
}

tbody tr:last-child td {
  border-bottom: none;
}
.cadastrar {
  margin-top: 12px;
  margin-bottom: 10px;
}
.grid-exportar {
  display: flex;
  justify-content: center;
}
.exportar {
  margin-bottom: 15px;
}
.filtrar {
  font-family: "Roboto", sans-serif;
  font-style: normal;
  font-size: 1.1rem;
  margin-bottom: 4px;
}
.aprovado {
  color: #28a745;
  font-weight: bold;
}

.reprovado {
  color: #dc3545;
  font-weight: bold;
}

.emExame {
  color: #ffc107;
  font-weight: bold;
}

.editar {
  margin-right: 8px;
}
</style>