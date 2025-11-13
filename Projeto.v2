from abc import ABC, abstractproperty, abstractclassmethod
from datetime import datetime


class Cliente:
  def __init__(self, endereco):
    self.endereco = endereco
    self.contas = []

  def realizar_transacao(self, conta, transacao):
    transacao.registrar(conta)

  def adicionar_conta(self, conta):
    self.contas.append(conta)


class PessoaFisica(Cliente):
  def __init__(self, nome, data_nascimento,cpf, endereco):
    super().__init__(endereco)
    self.cpf = cpf
    self.nome = nome
    self.data_nascimento = data_nascimento


class Conta:
  def __init__(self, numero, cliente):
    self._saldo = 0
    self._numero = numero
    self._agencia = '0001'
    self._cliente = cliente
    self._historico = Historico()

  @classmethod
  def nova_conta(cls, cliente, numero):
    return cls(numero, cliente)

  @property
  def saldo(self):
    return self._saldo

  @property
  def numero(self):
    return self._numero

  @property
  def agencia(self):
    return self._agencia

  @property
  def cliente(self):
    return self._cliente

  @property
  def historico(self):
    return self._historico

  def sacar(self, valor):
    saldo = self.saldo

    if valor > saldo:
      print('Você não tem saldo o suficiente!')
    elif valor > 0:
      self._saldo -= valor
      print('Saque realizado com sucesso!')
      return True
    else:
      print('O valor informado não é válido.')
    return False

  def depositar(self, valor):
    if valor > 0:
      self._saldo += valor
      print('Deposito realizado com sucesso!')
      return True
    else:
      print('O valor informado não é válido.')
      return False


class ContaCorrente(Conta):
  def __init__(self, numero, cliente, limite=1000, limite_saques=3):
    super().__init__(numero, cliente)
    self._limite = limite
    self._limite_saques = limite_saques

  def sacar(self, valor):
    numero_saques = len([transacao for transacao in self.historico.transacoes if transacao['tipo'] == Saque.__name__])

    if valor > self._limite:
      print('O valor do saque excede o limite')
    elif numero_saques >= self._limite_saques:
      print('Número máximo de saques alcançado')
    else:
      return super().sacar(valor)

    return False

  def __str__(self):
    return f"""\n
    Agência: \t{self.agencia}
    C/C:\t{self.numero}
    Titular:\t{self.cliente.nome}
    """


class Historico:
  def __init__(self):
    self._transacoes = []

  @property
  def transacoes(self):
    return self._transacoes

  def adicionar_transacao(self, transacao):
    self._transacoes.append({'tipo': transacao.__class__.__name__,
                             'valor': transacao.valor,
                             'data': datetime.now().strftime('%d-%m-%Y %H:%M:%S'),
                             })


class Transacao(ABC):
  @property
  @abstractproperty
  def valor(self):
    pass

  @abstractclassmethod
  def registrar(self, conta):
    pass


class Saque(Transacao):
  def __init__(self, valor):
    self._valor = valor

  @property
  def valor(self):
    return self._valor

  def registrar(self, conta):
    sucesso_transacao = conta.sacar(self.valor)

    if sucesso_transacao:
      conta.historico.adicionar_transacao(self)


class Deposito(Transacao):
  def __init__(self, valor):
    self._valor =  valor

  @property
  def valor(self):
    return self._valor

  def registrar(self, conta):
    sucesso_transacao = conta.depositar(self.valor)

    if sucesso_transacao:
      conta.historico.adicionar_transacao(self)


def menu():
  menu = '''\n
  ========== MENU ==========
  [d]\tDepositar
  [s]\tSacar
  [e]\tExtrato
  [nc]\tNova Conta
  [lc]\tListar Contas
  [nu]\tNovo Usuário
  [q]\tSair
  '''
  return input(menu)


def filtrar_cliente(cpf, clientes):
  clientes_filtrados = [cliente for cliente in clientes if cliente.cpf == cpf]
  return clientes_filtrados[0] if clientes_filtrados else None


def recuperar_conta(cliente):
  if not cliente.contas:
    print('Não possui conta.')
    return
  return cliente.contas[0]


def sacar(clientes):
  cpf = input('Digite o CPF: ')
  cliente = filtrar_cliente(cpf, clientes)

  if not cliente:
    print('Cliente não encontrado!')
    return

  valor = float(input('Informe o valor do saque: '))
  transacao = Saque(valor)
  conta = recuperar_conta(cliente)
  if not cliente:
    return
  cliente.realizar_transacao(conta, transacao)


def depositar(clientes):
  cpf = input('Digite o CPF: ')
  cliente = filtrar_cliente(cpf, clientes)

  if not cliente:
    print('Cliente não encontrado!')
    return

  valor = float(input('Informe o valor do depósito: '))
  transacao = Deposito(valor)
  conta = recuperar_conta(cliente)
  if not cliente:
    return
  cliente.realizar_transacao(conta, transacao)


def exibir_extrato(clientes):
  cpf = input('Digite o CPF: ')
  cliente = filtrar_cliente(cpf, clientes)

  if not cliente:
    print('Cliente não encontrado!')
    return

  conta = recuperar_conta(cliente)
  if not cliente:
    return

  transacoes = conta.historico.transacoes
  extrato = ''
  if not transacoes:
    extrato = 'Não foi realizada nenhuma transação.'
  else:
    for transacao in transacoes:
      extrato += f"\n{transacao['tipo']}:\n\tR${transacao['valor']:.2f}"


  print('\n========== EXTRATO ==========')
  print(extrato)
  print(f'\nSaldo:\n\tR${conta.saldo:.2f}')
  print('=============================')


def criar_cliente(clientes):
  cpf = input('Digite o CPF: ')
  cliente = filtrar_cliente(cpf, clientes)

  if cliente:
    print('Já existe um cliente com esse CPF.')
    return

  nome = input('Digite o nome completo: ')
  endereco = input('Digite o endereço (logradouro, nro - bairro - cidade/sigla do estado): ')
  data_nascimento = input('Digite a data de nascimento (dd/mm/aaaa): ')
  cliente = PessoaFisica(nome=nome, data_nascimento=data_nascimento, cpf=cpf, endereco=endereco)
  clientes.append(cliente)
  print('Cliente criado com sucesso!')

def criar_conta(clientes, numero_conta, contas):
  cpf = input('Digite o CPF: ')
  cliente = filtrar_cliente(cpf, clientes)

  if not cliente:
    print('Cliente não encontrado.')
    return

  conta = ContaCorrente.nova_conta(cliente=cliente, numero=numero_conta)
  contas.append(conta)
  cliente.contas.append(conta)
  print('conta criada com sucesso!')


def listar_contas(contas):
  for conta in contas:
     print('='*20)
     print(conta)

def main():
  clientes = []
  contas = []

  while True:
    opcao = menu()

    if opcao == 'd':
      depositar(clientes)

    elif opcao == 's':
      sacar(clientes)

    elif opcao == 'e':
      exibir_extrato(clientes)

    elif opcao == 'nu':
      criar_cliente(clientes)

    elif opcao == 'nc':
      numero_conta = len(contas) + 1
      criar_conta(clientes, numero_conta, contas)

    elif opcao == 'lc':
      listar_contas(contas)

    elif opcao == 'q':
      break

    else:
      print('Opcão inválida, selecione novamente a opção desejada.')


main()
