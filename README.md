# aula60
Aula60
package servidor

import (
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"github.com/gorilla/mux"
)

type usuario struct {
	Id uint32 `json: "id"`
	Nome string `json:"nome"`
	Email string `json:"email"` 
}

func CriarUsuario(w http.ResponseWriter, r *http.Request) {
	corpoRequisicao, erro := io.ReadAll(r.Body)
	if erro != nil {
		w.Write([]byte ("Falha ao ler o corpo da requisição"))
		return
	}

	var usuario usuario 
	if erro = json.Unmarshal(corpoRequisicao, &usuario); erro != nil {
		w.Write([]byte ("Erro ao converter o usuário para struct"))
		return
	}

	db, erro := banco.Conectar()
	if erro != nil {
		w.Write([]byte("Erro ao converter conectar no banco de dados!"))
		return
	}
	defer db.Close()
	

	statement, erro := db.Prepare("insert into usuarios (nome, email)values (?, ?)")
	if erro != nil {
		w.Write([]byte ("Erro ao criar o statement"))
		return
}
defer statement.Close()

insercao, erro := statement.Exec(usuario.Nome, usuario.Email)
if erro != nil {
	w.Write([]byte ("Erro ao executar o statement"))
	return

}

idInserido, erro := insercao.LastInsertId()
if erro != nil {
	w.Write([]byte ("Erro ao obert o id inserido"))
	return
}

// STATUS CODES
w.WriteHeader(http.StatusCreated)
w.Write([]byte(fmt.Sprintf("Usuário inserido com sucesso! Id: %d", idInserido)))

}
