# testes 

- vamos escrever os testes para nosso UsersService.
Para isso, dentro da pasta users vamos criar o arquivo users.service.spec.ts.

```
import { Test, TestingModule } from '@nestjs/testing';
import { UserRepository } from './../users/users.repository';
import { UsersService } from './users.service';

const mockUserRepository = () => ({
  createUser: jest.fn(),
  findOne: jest.fn(),
  delete: jest.fn(),
  findUsers: jest.fn(),
});

describe('UsersService', () => {
  let userRepository;
  let service;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useFactory: mockUserRepository,
        },
      ],
    }).compile();

    userRepository = await module.get<UserRepository>(UserRepository);
    service = await module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
    expect(userRepository).toBeDefined();
  });
});
```

- codigo comentado

```diff
import { Test, TestingModule } from '@nestjs/testing';
import { UserRepository } from './../users/users.repository';
import { UsersService } from './users.service';


+ mockUserRepository: Ele será responsável por simular todas as chamadas ao userRepository dentro do users.service.ts. Todos os métodos que teremos que emular dentro de nosso arquivo de testes foram especificados com o valor jest.fn(). Essa funcionalidade do jest permite emular o funcionamento de métodos externos à classe que estamos testando. Isso é importante para podermos separar os escopos dos nossos testes.

const mockUserRepository = () => ({
  createUser: jest.fn(),
  findOne: jest.fn(),
  delete: jest.fn(),
  findUsers: jest.fn(),
});

+ describe: Utilizamos o describe para separar as partes de nosso teste, sendo que podemos iniciar um describe dentro de outro de forma aninhada. No caso criaremos um describe para cada método do nosso UsersService.

describe('UsersService', () => {
  let userRepository;
  let service;

+ beforeEach: é uma função que será executada antes de cada teste a ser realizado. No caso utilizamos ela para inicializar nosso módulo e suas dependências que serão utilizadas durante os testes.

  beforeEach(async () => {
+ TestingModule, Test.createTestingModule: Esses recursos são naturais do próprio NestJS para serem utilizados em conjunto com o jest. Eles simulam a inicialização de um módulo e suas dependências. É importante aproveitarmos essa parte para inicializarmos o UserRepository que o UsersService precisa para funcionar com nosso mockUserRepository que irá simular as funcionalidades do UserRepository normal.

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useFactory: mockUserRepository,
        },
      ],
    }).compile();

    userRepository = await module.get<UserRepository>(UserRepository);
    service = await module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
    expect(userRepository).toBeDefined();
  });
});


```

- users.service.spec.ts
```diff
import { Test, TestingModule } from '@nestjs/testing';
import { UserRepository } from './../users/users.repository';
import { UsersService } from './users.service';
import { UserRole } from './user-roles.enum';
import { CreateUserDto } from './dto/create-user.dto';
import { UnprocessableEntityException } from '@nestjs/common';

const mockUserRepository = () => ({
  createUser: jest.fn(),
  findOne: jest.fn(),
  delete: jest.fn(),
  findUsers: jest.fn(),
});

describe('UsersService', () => {
  let userRepository;
  let service;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useFactory: mockUserRepository,
        },
      ],
    }).compile();

    userRepository = await module.get<UserRepository>(UserRepository);
    service = await module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
    expect(userRepository).toBeDefined();
  });

  describe('createUser', () => {
    let mockCreateUserDto: CreateUserDto;

    beforeEach(() => {
      mockCreateUserDto = {
        email: 'mock@email.com',
        name: 'Mock User',
        password: 'mockPassword',
        passwordConfirmation: 'mockPassword',
      };
    });

+ it: O it é utilizado para descrevermos cada teste que será realizado. No caso o primeiro teste que realizamos é se o UsersService e o UserRepository foram inicializados com sucesso.


    it('should create an user if passwords match', async () => {
      userRepository.createUser.mockResolvedValue('mockUser');
      const result = await service.createAdminUser(mockCreateUserDto);

+ expect: é através do expect que verificamos se as coisas aconteceram como esperado. Nesse primeiro teste nós esperávamos que o service e o userRepository fossem inicializados com sucesso durante a criação do módulo de teste, por isso esperávamos que seus valores fossem definidos, ou seja, diferentes de undefined.

      expect(userRepository.createUser).toHaveBeenCalledWith(
        mockCreateUserDto,
        UserRole.ADMIN,
      );
      expect(result).toEqual('mockUser');
    });

    it('should throw an error if passwords doesnt match', async () => {
      mockCreateUserDto.passwordConfirmation = 'wrongPassword';
      expect(service.createAdminUser(mockCreateUserDto)).rejects.toThrow(
        UnprocessableEntityException,
      );
    });
  });
});

```

>Basicamente serão essas funcionalidades que utilizaremos no decorrer dos testes para verificar se nosso código funciona como esperado. Vamos começar testando nosso método createUser:

- users.service.spec.ts

``iff
import { Test, TestingModule } from '@nestjs/testing';
import { UserRepository } from './../users/users.repository';
import { UsersService } from './users.service';
import { UserRole } from './user-roles.enum';
import { CreateUserDto } from './dto/create-user.dto';
import { UnprocessableEntityException } from '@nestjs/common';

const mockUserRepository = () => ({
  createUser: jest.fn(),
  findOne: jest.fn(),
  delete: jest.fn(),
  findUsers: jest.fn(),
});

describe('UsersService', () => {
  let userRepository;
  let service;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useFactory: mockUserRepository,
        },
      ],
    }).compile();

    userRepository = await module.get<UserRepository>(UserRepository);
    service = await module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
    expect(userRepository).toBeDefined();
  });

  describe('createUser', () => {
    let mockCreateUserDto: CreateUserDto;

    beforeEach(() => {
      mockCreateUserDto = {
        email: 'mock@email.com',
        name: 'Mock User',
        password: 'mockPassword',
        passwordConfirmation: 'mockPassword',
      };
    });

    it('should create an user if passwords match', async () => {
      userRepository.createUser.mockResolvedValue('mockUser');
      const result = await service.createAdminUser(mockCreateUserDto);

      expect(userRepository.createUser).toHaveBeenCalledWith(
        mockCreateUserDto,
        UserRole.ADMIN,
      );
      expect(result).toEqual('mockUser');
    });

    it('should throw an error if passwords doesnt match', async () => {
      mockCreateUserDto.passwordConfirmation = 'wrongPassword';
      expect(service.createAdminUser(mockCreateUserDto)).rejects.toThrow(
        UnprocessableEntityException,
      );
    });
  });
});

```

Reparem que construímos os testes do método createUser dentro do describe do UsersService, que é onde o método se encontra.

Vamos analisar o que está acontecendo:

- Utilizamos o beforeEach para inicializarmos o valor do mockCreateUserDto, uma vez que esse é o argumento esperado pelo método createAdminUser e dentro dos nossos testes queremos ter controle sobre a entrada do método.
- O primeiro caso de testes que analisaremos será se o usuário será criado caso não tenha nenhum problema. O jest.fn() nos dá acesso ao método mockResolvedValue que simula uma execução bem sucedida de uma função assíncrona (no userRepository a função createUser é assíncrona). Vamos simular que, caso a execução seja bem sucedida, o método irá nos retornar simplesmente a string ‘mockUser’.
- Realizamos a chamada ao método que queremos testar com o valor de testes mockCreateUserDto.
- Após a execução do método createAdminUser esperamos que algumas coisas tenham acontecido. A primeira é que o método createUser do UserRepository tenha sido chamado especificamente com os argumentos mockCreateUserDto e UserRole.ADMIN. Após isso esperamos que o método tenha sido executado com sucesso e que tenha nos retornado a string ‘mockUser’ conforme especificamos no início do teste que uma execução bem sucedida do createUser irá retornar esse valor. 
>Lembrando que isso só é possível pois inicializamos esse método com o valor jest.fn() na criação do mockUserRepository.

## segundo caso de testes: O método createAdminUser deve retornar um erro caso as senhas sejam diferentes:

- Primeiro nós atribuímos um valor ao campo passwordConfirmation que seja diferente do valor do campo password.
- Em seguida nós simulamos a execução do método createAdminUser com um mockCreateUserDto onde as senhas não coincidem. O resultado aguardado é que a Promise seja rejeitada com o erro UnprocessableEntityException.

- Podemos executar nossos testes com o comando:
```
npm run test:watch
```

se tudo correr bem sera exibido passed

## Agora podemos testar o método findUserById:

```diff
import { Test, TestingModule } from '@nestjs/testing';
import { UserRepository } from './../users/users.repository';
import { UsersService } from './users.service';
import { UserRole } from './user-roles.enum';
import { CreateUserDto } from './dto/create-user.dto';
import {
  UnprocessableEntityException,
  NotFoundException,
} from '@nestjs/common';

const mockUserRepository = () => ({
  createUser: jest.fn(),
  findOne: jest.fn(),
  delete: jest.fn(),
  findUsers: jest.fn(),
});

describe('UsersService', () => {
  let userRepository;
  let service;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useFactory: mockUserRepository,
        },
      ],
    }).compile();

    userRepository = await module.get<UserRepository>(UserRepository);
    service = await module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
    expect(userRepository).toBeDefined();
  });

  describe('createUser', () => {
    let mockCreateUserDto: CreateUserDto;

    beforeEach(() => {
      mockCreateUserDto = {
        email: 'mock@email.com',
        name: 'Mock User',
        password: 'mockPassword',
        passwordConfirmation: 'mockPassword',
      };
    });

    it('should create an user if passwords match', async () => {
      userRepository.createUser.mockResolvedValue('mockUser');
      const result = await service.createAdminUser(mockCreateUserDto);

      expect(userRepository.createUser).toHaveBeenCalledWith(
        mockCreateUserDto,
        UserRole.ADMIN,
      );
      expect(result).toEqual('mockUser');
    });

    it('should throw an error if passwords doesnt match', async () => {
      mockCreateUserDto.passwordConfirmation = 'wrongPassword';
      expect(service.createAdminUser(mockCreateUserDto)).rejects.toThrow(
        UnprocessableEntityException,
      );
    });
  });

  describe('findUserById', () => {
    it('should return the found user', async () => {
      userRepository.findOne.mockResolvedValue('mockUser');
      expect(userRepository.findOne).not.toHaveBeenCalled();

      const result = await service.findUserById('mockId');
      const select = ['email', 'name', 'role', 'id'];
      expect(userRepository.findOne).toHaveBeenCalledWith('mockId', { select });
      expect(result).toEqual('mockUser');
    });

    it('should throw an error as user is not found', async () => {
      userRepository.findOne.mockResolvedValue(null);
      expect(service.findUserById('mockId')).rejects.toThrow(NotFoundException);
    });
  });
});

```

- Criamos um novo describe para os testes do findUserById.
- O primeiro teste realizado é caso um ID válido seja passado ao método.
- Especificamos a string ‘mockUser’ como o valor de retorno do método findOne do UserRepository caso o método seja executado com sucesso.
- Nós esperamos que, até esse momento, o método findOne ainda não tenha sido executado nenhuma vez.
- Simulamos a execução do método findUserById com um ID qualquer.
- Especificamos os campos que serão passados como parâmetro para o select do método findOne do UserRepository.
- Após a execução do método findUserById do nosso UsersService nós esperamos que o método findOne do UserRepository tenha sido executado com os argumentos ‘mockId’ e { select } sendo select o valor que definimos anteriormente.
- Nós também esperamos que o valor retornado após a execução bem sucedida de findOnde seja a string ‘mockUser’.

## o segundo teste que realizamos, que é o caso do método findOne do UserRepository não ter encontrado um usuário a partir do ID informado:

- o segundo teste que realizamos, que é o caso do método findOne do UserRepository não ter encontrado um usuário a partir do ID informado:
- Depois simulamos a execução do método findUserById, onde esperamos que a função retorne o erro NotFoundException.

## O próximo método da lista é o updateUsers, entretamos teremos que realizar algumas modificações nele para que ele se torne testável. Vamos pular ele por enquanto e seguir para o deleteUser:

```diff
import { Test, TestingModule } from '@nestjs/testing';
import { UserRepository } from './../users/users.repository';
import { UsersService } from './users.service';
import { UserRole } from './user-roles.enum';
import { CreateUserDto } from './dto/create-user.dto';
import {
  UnprocessableEntityException,
  NotFoundException,
} from '@nestjs/common';

const mockUserRepository = () => ({
  createUser: jest.fn(),
  findOne: jest.fn(),
  delete: jest.fn(),
  findUsers: jest.fn(),
});

describe('UsersService', () => {
  let userRepository;
  let service;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useFactory: mockUserRepository,
        },
      ],
    }).compile();

    userRepository = await module.get<UserRepository>(UserRepository);
    service = await module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
    expect(userRepository).toBeDefined();
  });

  describe('createUser', () => {
    let mockCreateUserDto: CreateUserDto;

    beforeEach(() => {
      mockCreateUserDto = {
        email: 'mock@email.com',
        name: 'Mock User',
        password: 'mockPassword',
        passwordConfirmation: 'mockPassword',
      };
    });

    it('should create an user if passwords match', async () => {
      userRepository.createUser.mockResolvedValue('mockUser');
      const result = await service.createAdminUser(mockCreateUserDto);

      expect(userRepository.createUser).toHaveBeenCalledWith(
        mockCreateUserDto,
        UserRole.ADMIN,
      );
      expect(result).toEqual('mockUser');
    });

    it('should throw an error if passwords doesnt match', async () => {
      mockCreateUserDto.passwordConfirmation = 'wrongPassword';
      expect(service.createAdminUser(mockCreateUserDto)).rejects.toThrow(
        UnprocessableEntityException,
      );
    });
  });

  describe('findUserById', () => {
    it('should return the found user', async () => {
      userRepository.findOne.mockResolvedValue('mockUser');
      expect(userRepository.findOne).not.toHaveBeenCalled();

      const result = await service.findUserById('mockId');
      const select = ['email', 'name', 'role', 'id'];
      expect(userRepository.findOne).toHaveBeenCalledWith('mockId', { select });
      expect(result).toEqual('mockUser');
    });

    it('should throw an error as user is not found', async () => {
      userRepository.findOne.mockResolvedValue(null);
      expect(service.findUserById('mockId')).rejects.toThrow(NotFoundException);
    });
  });

  describe('deleteUser', () => {
    it('should return affected > 0 if user is deleted', async () => {
      userRepository.delete.mockResolvedValue({ affected: 1 });

      await service.deleteUser('mockId');
      expect(userRepository.delete).toHaveBeenCalledWith({ id: 'mockId' });
    });

    it('should throw an error if no user is deleted', async () => {
      userRepository.delete.mockResolvedValue({ affected: 0 });

      expect(service.deleteUser('mockId')).rejects.toThrow(NotFoundException);
    });
  });
});

```

- Criamos um describe para os testes do método deleteUser
- Testamos uma execução bem sucedida do método, onde o método delete do UserRepository nos retorna affected > 0.
- Testamos uma execução falha do método, onde o método delete do UserRepository nos retorna affected = 0 e então o método deleteUser retorna um erro do tipo NotFoundException indicando que não foi encontrado um usuário com o ID informado para ser deletado.

## O próximo teste será para o método findUsers, que será o teste mais simples de todos:

```diff
import { Test, TestingModule } from '@nestjs/testing';
import { UserRepository } from './../users/users.repository';
import { UsersService } from './users.service';
import { UserRole } from './user-roles.enum';
import { CreateUserDto } from './dto/create-user.dto';
import {
  UnprocessableEntityException,
  NotFoundException,
} from '@nestjs/common';
import { FindUsersQueryDto } from './dto/find-users-query-dto';

const mockUserRepository = () => ({
  createUser: jest.fn(),
  findOne: jest.fn(),
  delete: jest.fn(),
  findUsers: jest.fn(),
});

describe('UsersService', () => {
  let userRepository;
  let service;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useFactory: mockUserRepository,
        },
      ],
    }).compile();

    userRepository = await module.get<UserRepository>(UserRepository);
    service = await module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
    expect(userRepository).toBeDefined();
  });

  describe('createUser', () => {
    let mockCreateUserDto: CreateUserDto;

    beforeEach(() => {
      mockCreateUserDto = {
        email: 'mock@email.com',
        name: 'Mock User',
        password: 'mockPassword',
        passwordConfirmation: 'mockPassword',
      };
    });

    it('should create an user if passwords match', async () => {
      userRepository.createUser.mockResolvedValue('mockUser');
      const result = await service.createAdminUser(mockCreateUserDto);

      expect(userRepository.createUser).toHaveBeenCalledWith(
        mockCreateUserDto,
        UserRole.ADMIN,
      );
      expect(result).toEqual('mockUser');
    });

    it('should throw an error if passwords doesnt match', async () => {
      mockCreateUserDto.passwordConfirmation = 'wrongPassword';
      expect(service.createAdminUser(mockCreateUserDto)).rejects.toThrow(
        UnprocessableEntityException,
      );
    });
  });

  describe('findUserById', () => {
    it('should return the found user', async () => {
      userRepository.findOne.mockResolvedValue('mockUser');
      expect(userRepository.findOne).not.toHaveBeenCalled();

      const result = await service.findUserById('mockId');
      const select = ['email', 'name', 'role', 'id'];
      expect(userRepository.findOne).toHaveBeenCalledWith('mockId', { select });
      expect(result).toEqual('mockUser');
    });

    it('should throw an error as user is not found', async () => {
      userRepository.findOne.mockResolvedValue(null);
      expect(service.findUserById('mockId')).rejects.toThrow(NotFoundException);
    });
  });

  describe('deleteUser', () => {
    it('should return affected > 0 if user is deleted', async () => {
      userRepository.delete.mockResolvedValue({ affected: 1 });

      await service.deleteUser('mockId');
      expect(userRepository.delete).toHaveBeenCalledWith({ id: 'mockId' });
    });

    it('should throw an error if no user is deleted', async () => {
      userRepository.delete.mockResolvedValue({ affected: 0 });

      expect(service.deleteUser('mockId')).rejects.toThrow(NotFoundException);
    });
  });

  describe('findUsers', () => {
    it('should call the findUsers method of the userRepository', async () => {
      userRepository.findUsers.mockResolvedValue('resultOfsearch');
      const mockFindUsersQueryDto: FindUsersQueryDto = {
        name: '',
        email: '',
        limit: 1,
        page: 1,
        role: '',
        sort: '',
        status: true,
      };
      const result = await service.findUsers(mockFindUsersQueryDto);
      expect(userRepository.findUsers).toHaveBeenCalledWith(
        mockFindUsersQueryDto,
      );
      expect(result).toEqual('resultOfsearch');
    });
  });
});

```

- O único teste que temos que realizar aqui é se o método findUsers do UserRepository foi chamado com o valor passado como parâmetro para o método findUsers do UsersService, nada de mais.

## Por último vamos aos testes do updateUser. Primeiro vamos realizar as alterações ao método que comentei anteriormente. Para que o método se torne testável, vamos reescrever ele da seguinte forma:
> updateUser.ts
```
async updateUser(updateUserDto: UpdateUserDto, id: string) {
  const result = await this.userRepository.update({ id }, updateUserDto);
  if (result.affected > 0) {
    const user = await this.findUserById(id);
    return user;
  } else {
    throw new NotFoundException('Usuário não encontrado');
  }
}
```

>users.service.ts
```
import {
  Injectable,
  UnprocessableEntityException,
  NotFoundException,
} from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { UserRepository } from './users.repository';
import { CreateUserDto } from './dto/create-user.dto';
import { User } from './user.entity';
import { UserRole } from './user-roles.enum';
import { UpdateUserDto } from './dto/update-user.dto';
import { FindUsersQueryDto } from './dto/find-users-query-dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(UserRepository)
    private userRepository: UserRepository,
  ) {}

  async createAdminUser(createUserDto: CreateUserDto): Promise<User> {
    if (createUserDto.password != createUserDto.passwordConfirmation) {
      throw new UnprocessableEntityException('As senhas não conferem');
    } else {
      return this.userRepository.createUser(createUserDto, UserRole.ADMIN);
    }
  }

  async findUserById(userId: string): Promise<User> {
    const user = await this.userRepository.findOne(userId, {
      select: ['email', 'name', 'role', 'id'],
    });

    if (!user) throw new NotFoundException('Usuário não encontrado');

    return user;
  }

  async updateUser(updateUserDto: UpdateUserDto, id: string) {
    const result = await this.userRepository.update({ id }, updateUserDto);
    if (result.affected > 0) {
      const user = await this.findUserById(id);
      return user;
    } else {
      throw new NotFoundException('Usuário não encontrado');
    }
  }

  async deleteUser(userId: string) {
    const result = await this.userRepository.delete({ id: userId });
    if (result.affected === 0) {
      throw new NotFoundException(
        'Não foi encontrado um usuário com o ID informado',
      );
    }
  }

  async findUsers(
    queryDto: FindUsersQueryDto,
  ): Promise<{ users: User[]; total: number }> {
    const users = await this.userRepository.findUsers(queryDto);
    return users;
  }
}
```

- Antes nós buscávamos o usuário no banco de dados com o ID informado verificando sua existência, verificávamos os campos informados no updateUserDto para atualizar os respectivos campos no usuário encontrado e faziamos uma segunda chamada ao banco de dados para atualizar os dados.

- Agor nós realizamos uma chamada ao banco de dados para atualizar os dados, delegando a lógica para o método update() já padrão de um repositório TypeORM e verificamos se houve algum registro afetado no banco de dados. Se não houve um registro afetado nós retornamos um erro de que o ID é inválido. Se houve algum registro afetado significa que o ID é válido e os dados foram alterados com sucesso. Em seguida realizamos uma segunda chamada ao banco de dados para encontrar o usuário alterado, com os dados já atualizados, para podermos retornar o mesmo para o front, da mesma forma que faziamos anteriormente.

> Observem que continuamos fazendo apenas duas chamadas ao banco de dados, igual anteriormente, então nossa performânce de modo geral não foi significativamente afetada.

## Vamos agora escrever os testes para esse método:

```
import { Test, TestingModule } from '@nestjs/testing';
import { UserRepository } from './../users/users.repository';
import { UsersService } from './users.service';
import { UserRole } from './user-roles.enum';
import { CreateUserDto } from './dto/create-user.dto';
import {
  UnprocessableEntityException,
  NotFoundException,
} from '@nestjs/common';
import { FindUsersQueryDto } from './dto/find-users-query-dto';

const mockUserRepository = () => ({
  createUser: jest.fn(),
  findOne: jest.fn(),
  delete: jest.fn(),
  findUsers: jest.fn(),
  update: jest.fn(),
});

describe('UsersService', () => {
  let userRepository;
  let service;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useFactory: mockUserRepository,
        },
      ],
    }).compile();

    userRepository = await module.get<UserRepository>(UserRepository);
    service = await module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
    expect(userRepository).toBeDefined();
  });

  describe('createUser', () => {
    let mockCreateUserDto: CreateUserDto;

    beforeEach(() => {
      mockCreateUserDto = {
        email: 'mock@email.com',
        name: 'Mock User',
        password: 'mockPassword',
        passwordConfirmation: 'mockPassword',
      };
    });

    it('should create an user if passwords match', async () => {
      userRepository.createUser.mockResolvedValue('mockUser');
      const result = await service.createAdminUser(mockCreateUserDto);

      expect(userRepository.createUser).toHaveBeenCalledWith(
        mockCreateUserDto,
        UserRole.ADMIN,
      );
      expect(result).toEqual('mockUser');
    });

    it('should throw an error if passwords doesnt match', async () => {
      mockCreateUserDto.passwordConfirmation = 'wrongPassword';
      expect(service.createAdminUser(mockCreateUserDto)).rejects.toThrow(
        UnprocessableEntityException,
      );
    });
  });

  describe('findUserById', () => {
    it('should return the found user', async () => {
      userRepository.findOne.mockResolvedValue('mockUser');
      expect(userRepository.findOne).not.toHaveBeenCalled();

      const result = await service.findUserById('mockId');
      const select = ['email', 'name', 'role', 'id'];
      expect(userRepository.findOne).toHaveBeenCalledWith('mockId', { select });
      expect(result).toEqual('mockUser');
    });

    it('should throw an error as user is not found', async () => {
      userRepository.findOne.mockResolvedValue(null);
      expect(service.findUserById('mockId')).rejects.toThrow(NotFoundException);
    });
  });

  describe('deleteUser', () => {
    it('should return affected > 0 if user is deleted', async () => {
      userRepository.delete.mockResolvedValue({ affected: 1 });

      await service.deleteUser('mockId');
      expect(userRepository.delete).toHaveBeenCalledWith({ id: 'mockId' });
    });

    it('should throw an error if no user is deleted', async () => {
      userRepository.delete.mockResolvedValue({ affected: 0 });

      expect(service.deleteUser('mockId')).rejects.toThrow(NotFoundException);
    });
  });

  describe('findUsers', () => {
    it('should call the findUsers method of the userRepository', async () => {
      userRepository.findUsers.mockResolvedValue('resultOfsearch');
      const mockFindUsersQueryDto: FindUsersQueryDto = {
        name: '',
        email: '',
        limit: 1,
        page: 1,
        role: '',
        sort: '',
        status: true,
      };
      const result = await service.findUsers(mockFindUsersQueryDto);
      expect(userRepository.findUsers).toHaveBeenCalledWith(
        mockFindUsersQueryDto,
      );
      expect(result).toEqual('resultOfsearch');
    });
  });

  describe('updateUser', () => {
    it('should return affected > 0 if user data is updated and return the new user', async () => {
      userRepository.update.mockResolvedValue({ affected: 1 });
      userRepository.findOne.mockResolvedValue('mockUser');

      const result = await service.updateUser('mockUpdateUserDto', 'mockId');
      expect(userRepository.update).toHaveBeenCalledWith(
        { id: 'mockId' },
        'mockUpdateUserDto',
      );
      expect(result).toEqual('mockUser');
    });

    it('should throw an error if no row is affected in the DB', async () => {
      userRepository.update.mockResolvedValue({ affected: 0 });

      expect(service.updateUser('mockUpdateUserDto', 'mockId')).rejects.toThrow(
        NotFoundException,
      );
    });
  });
});

```
>Podemos observar que os testes para o método updateUser ficaram semelhantes aos do método deleteUser, com algumas pequenas excessões:

- Dentro de updateUser, caso algum dado no banco tenha sido afetado (affected > 0), realizamos uma chamada à função findUserById, então é necessário especificar um valor de retorno para a função findOne do UserRepository, uma vez que a mesma é chamada dentro de findUserById. Então nós verificamos se userRepository.update foi chamada com os argumentos corretos e posteriormente se o método updateUser do USersService retornou o usuário corretamente.

- Dentro de updateUser, caso algum dado no banco tenha sido afetado (affected > 0), realizamos uma chamada à função findUserById, então é necessário especificar um valor de retorno para a função findOne do UserRepository, uma vez que a mesma é chamada dentro de findUserById. Então nós verificamos se userRepository.update foi chamada com os argumentos corretos e posteriormente se o método updateUser do USersService retornou o usuário corretamente.O segundo teste, caso não tenha sido encontrado um usuário com o ID especificado, é semelhante ao do deleteUser, onde o método retorna um erro do tipo NotFoundException caso affected = 0.

>E com isso finalizamos nosso tutorial sobre testes. Lembrando que o ideal seria escrever testes para todo código que escrevemos até agora, entretanto eles não são tão diferentes do que vimos neste tutorial, o que tornaria esse texto demasiadamente longo e repetitivo se fossemos escrever todos os testes aqui. Em breve publico a última parte deste tutorial, mostrando como dar deploy da nossa API!