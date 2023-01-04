# Dica 1
    Para criar um arquivo mockado (ficticio) vazio devemos utilizar: 
    "const mockFiles = Object.create(null)"

    Para criar um arquivo mockado "cheio" devemos copiar exatamente como está as propriedades do objeto. Exemplo: um object tem as propriedades {altura,peso,idade}. O objeto mockado será assim:
    
    "const data = {
      altura: 1.85 ,
      peso: 100,
      idade: 25,
      file: mockFiles,
    };
