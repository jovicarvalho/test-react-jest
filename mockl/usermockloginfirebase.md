## Simulação de usuário Logado
Para simular um usuário logado, devemos importar o firebase app na nossa página de teste, juntamente com um provedor do google, um simulador de incialização do firebase e o google login:
    
    import useGoogleLogin from '.';

    import { googleProviderFunc, firebaseInitialize } from '../../firebase/config';
    import firebase from 'firebase/compat/app';

    jest.mock('../../firebase/config');

Logo após devemos mockar o provedor e mockar o inicializador
    
    const mockGoogleProviderFunc = googleProviderFunc as jest.MockedFunction<
    typeof googleProviderFunc
    >;

    const mockFirebaseInitialize = firebaseInitialize as jest.MockedFunction<
    typeof firebaseInitialize
    >;

    jest.mock('firebase/compat/app', () => ({
    initializeApp: jest.fn(),
    apps: ['app'],
    }));

Finalizando com o mock do usuário logado:

    mockGoogleProviderFunc.mockImplementation(() => {
        return {} as firebase.auth.GoogleAuthProvider;
      });
      mockFirebaseInitialize.mockReturnValue({
        app: {} as any,
        auth: {
          signInWithPopup: mockSignInWithPopup,
        } as any,
        googleProvider: jest.fn().mockReturnValue(true),
      });

Tendo tudo isso feito, o mock de autenticação ja esta finalizado os componentes podem ser testados
Alguns exemplos de código:
    import React from 'react';
import { act, render, screen } from '@testing-library/react';
import Home from './index';
import { GuideInterface, getGuides } from '@services/guides';
import { AxiosResponse } from 'axios';
import '@testing-library/jest-dom/extend-expect';

jest.mock('react-router-dom', () => {
  const useNavigate = jest.fn();
  return {
    useNavigate,
  };
});

jest.mock('@services/guides');

const getGuidesMock = getGuides as jest.MockedFunction<typeof getGuides>;

describe('Componente do Home', () => {
  test('deve existir um parágrafo na Home', () => {
    render(<Home />);
    
    const HomeDescription = screen.getByText(
      "Bem-vindo ao DB INCLUI, o DB INCLUI é um web app que dissemina a cultura de inclusão dentro da DBserver, com foco na cultura surda.",
    );
    expect(HomeDescription).toBeInTheDocument();
  });

  beforeEach(() => {
    getGuidesMock.mockClear();
  });

  test('Deve mostrar a página com os Guias', async () => {
    getGuidesMock.mockImplementation(
      async () =>
        ({
          data: { data: [] },
        } as unknown as Promise<AxiosResponse<{ data: GuideInterface[] }>>),
    );

    // eslint-disable-next-line testing-library/no-unnecessary-act
    await act(async () => {
      render(<Home />);
    });

    expect(getGuidesMock).toBeCalledTimes(1);
  });

  test('Quando o component for renderizado deve ser apresentado o conteúdo', async () => {
    const cardMock = {
      title: 'teste',
      content: 'teste',
      filePaths: {
        filePath: 'a',
        publicId: 'b',
      },
    };
    getGuidesMock.mockImplementation(
      async () =>
        ({
          data: { data: [cardMock, cardMock, cardMock, cardMock] },
        } as unknown as Promise<AxiosResponse<{ data: GuideInterface[] }>>),
    );

    // eslint-disable-next-line testing-library/no-unnecessary-act
    await act(async () => {
      render(<Home />);
    });

    expect(getGuidesMock).toBeCalledTimes(1);
  });

  test('Quando houver algum erro no serviço deve ser apresentado uma mensagem de erro', async () => {
    getGuidesMock.mockRejectedValue(true);
    // eslint-disable-next-line testing-library/no-unnecessary-act
    await act(async () => {
      render(<Home />);
    });
    expect(getGuidesMock).toBeCalledTimes(1);
    const errorMessage = 'Desculpe, ocorreu um erro ao carregar a página!';
    const erroMessageElement = screen.getByText(errorMessage);
    expect(erroMessageElement.textContent).toBe(errorMessage);
  });
});
