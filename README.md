Isso é normal no começo no **Qt Designer**. Vamos fazer **passo-a-passo exatamente onde clicar** para renomear os widgets.

---

# 1️⃣ Clique no widget primeiro

Abra:

```
mainwindow.ui
```

Agora:

1️⃣ Clique **uma vez** no widget que quer renomear.

Exemplo:

* clique no **campo de texto**
* ou no **botão**

Você verá **quadradinhos azuis ao redor** do elemento.

---

# 2️⃣ Abrir o painel de propriedades

No **lado direito da tela** existe o painel:

```
Property Editor
```

Se não estiver visível:

```
View → Views → Property Editor
```

---

# 3️⃣ Encontrar "objectName"

No painel direito procure:

```
QObject
```

Dentro dele existe:

```
objectName
```

Exemplo:

```
objectName: pushButton
```

---

# 4️⃣ Alterar o nome

Clique no valor e altere.

### Botão

```
pushButton  → sendButton
```

### Campo de texto

```
lineEdit → nameInput
```

### Label

```
label → nameLabel
```

---

# 5️⃣ Pressione ENTER

Depois de alterar o nome:

```
Enter
```

O Qt salva automaticamente.

---

# 📌 Outra forma (mais fácil)

No lado esquerdo existe:

```
Object Inspector
```

Lista assim:

```
MainWindow
   centralWidget
      label
      lineEdit
      pushButton
```

Você pode:

1️⃣ clicar com botão direito
2️⃣ escolher

```
Change objectName
```

---

# ⚠️ Problema comum

Se não deixa editar, normalmente é porque você está mudando **o texto** e não **o objectName**.

| Coisa      | O que faz            |
| ---------- | -------------------- |
| text       | texto que aparece    |
| objectName | nome usado no código |

---

# Exemplo

Botão:

```
text: Send
objectName: sendButton
```

---

# 📷 Se quiser

Mande **um print do Qt Creator** da tela do `mainwindow.ui`.

Eu consigo te mostrar **exatamente onde clicar**, porque o Qt às vezes muda a posição dos painéis dependendo da versão.
