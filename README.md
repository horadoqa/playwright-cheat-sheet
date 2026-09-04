# **Playwright Cheat Sheet**

Focado nos comandos mais usados com **JavaScript/TypeScript**.

 ## Instalação e execução

 | Comando | Descrição |
| --- | --- |
| `npm init playwright@latest` | Cria e configura um novo projeto Playwright |
| `npx playwright install` | Instala os navegadores necessários para o Playwright |
| `npx playwright install chormium` | Instala somente o navegador Chromium |
| `npx playwright install firefox` | Instala somente o navegador Firefox |
| `npx playwright install webkit` | Instala somente o navegador Webkit|
| `npx playwright uninstall` | Desinstala os navegadores |
| `npx playwright version` | Verifica a versão do Playwright |
| `npx playwright test` | Executa os testes |
| `npx playwright test --headed` | Executa os testes exibindo o navegador |
| `npx playwright test --debug` | Executa os testes no modo de depuração |
| `npx playwright test --ui` | Abre a interface gráfica do Playwright para executar e acompanhar os testes |
| `npx playwright show-report` | Exibe o relatório dos testes realizados |

 ## Estrutura básica

```
import { test, expect } from '@playwright/test';

test('meu teste', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveTitle(/Example/);
});
```

 ## Navegação

```
await page.goto('https://example.com');

await page.goBack();
await page.goForward();
await page.reload();

await page.waitForURL('**/dashboard');
```

 ## Locators

```
page.getByRole('button', { name: 'Login' });
page.getByText('Bem-vindo');
page.getByLabel('Email');
page.getByPlaceholder('Digite seu email');
page.getByTestId('submit-button');

page.locator('#username');
page.locator('.btn-primary');
page.locator('input[name="email"]');
```

 ## Ações

```
await page.getByRole('button', { name: 'Login' }).click();

await page.getByLabel('Email').fill('user@example.com');
await page.getByLabel('Senha').fill('123456');

await page.getByLabel('Nome').press('Enter');

await page.locator('input[type="checkbox"]').check();
await page.locator('input[type="checkbox"]').uncheck();

await page.selectOption('select', 'sp');

await page.locator('input[type="file"]').setInputFiles('arquivo.pdf');
```

 ## Expect

```
await expect(page).toHaveTitle('Página inicial');

await expect(page).toHaveURL(/dashboard/);

await expect(page.getByText('Sucesso')).toBeVisible();

await expect(page.getByRole('button')).toBeEnabled();

await expect(page.getByRole('button')).toBeDisabled();

await expect(page.getByText('Olá')).toHaveText('Olá, João');

await expect(page.locator('.item')).toHaveCount(5);
```

 ## Locators avançados

```
page.getByRole('button').filter({ hasText: 'Salvar' });

page.locator('.card').filter({
  hasText: 'Produto'
});

page.locator('.item').first();
page.locator('.item').last();
page.locator('.item').nth(2);

page.locator('button').and(page.getByText('Salvar'));
```

 ## Frames

```
const frame = page.frameLocator('#iframe');

await frame.getByRole('button', { name: 'Enviar' }).click();
```

 ## Abas / páginas

```
const newPagePromise = page.waitForEvent('popup');

await page.getByRole('link', { name: 'Abrir' }).click();

const newPage = await newPagePromise;

await newPage.waitForLoadState();
```

 ## Screenshots

```
await page.screenshot({
  path: 'screenshot.png'
});

await page.screenshot({
  path: 'full.png',
  fullPage: true
});
```

 ## Seletores CSS/XPath

```
await page.locator('#login').click();

await page.locator('.menu > li').click();

await page.locator('input[name="email"]').fill('teste@email.com');

await page.locator('xpath=//button[text()="Enviar"]').click();
```

 **Preferência:** `getByRole`, `getByLabel`, `getByText` e `getByTestId` geralmente tornam os testes mais robustos e legíveis.

 ## Esperas

 Evite, quando possível:

```
await page.waitForTimeout(3000);
```

 Prefira esperar pela condição:

```
await expect(page.getByText('Carregado')).toBeVisible();

await page.waitForURL('**/dashboard');

await page.waitForLoadState('networkidle');
```

 ## Interceptação de API

```
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 200,
    body: JSON.stringify([
      { id: 1, name: 'João' }
    ])
  });
});
```

 ## Mock de API

```
await page.route('**/api/login', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
      token: 'abc123'
    })
  });
});
```

 ## Requisições HTTP

```
test('API', async ({ request }) => {
  const response = await request.get('/api/users');

  expect(response.ok()).toBeTruthy();

  const data = await response.json();
});
```

 ## Cookies e storage

```
await context.addCookies([
  {
    name: 'session',
    value: 'abc123',
    domain: 'example.com',
    path: '/'
  }
]);
```

 Salvar estado:

```
npx playwright codegen https://example.com
```

 E usar um `storageState`:

```
test.use({
  storageState: 'auth.json'
});
```

 ## Debug

| Comando | Descrição |
| --- | --- |
| `npx playwright test --debug` | Executa os testes no modo de depuração, permitindo analisar cada etapa |
| `npx playwright test --headed` | Executa os testes com o navegador visível |
| `npx playwright test --ui` | Abre a interface gráfica do Playwright para executar e acompanhar os testes |

 Dentro do teste:

```
await page.pause();
```

 Gerar código automaticamente:

```
npx playwright codegen https://example.com
```

 ## Rodar testes específicos

| Comando | Descrição |
| --- | --- |
| `npx playwright test login.spec.ts` | Executa o arquivo de teste `login.spec.ts` |
| `npx playwright test tests/login.spec.ts` | Executa o arquivo de teste `login.spec.ts` localizado na pasta `tests` |
| `npx playwright test -g "login"` | Executa apenas os testes que correspondem ao nome ou descrição `"login"` |
| `npx playwright test --project=chromium` | Executa os testes utilizando o projeto/navegador Chromium |
| `npx playwright test --project=firefox` | Executa os testes utilizando o projeto/navegador Firefox |
| `npx playwright test --project=webkit` | Executa os testes utilizando o projeto/navegador Webkit |
| `npx playwright test --workers=1` | Executa os testes utilizando apenas 1 worker, de forma sequencial |
| `npx playwright test --repeat-each=3` | Executa cada teste 3 vezes |
| `npx playwright test --report=list` | Executa o teste com report list |
| `npx playwright test --report=html` | Gera o reporte HTML |

 ## Configuração (`playwright.config.ts`)

```
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  use: {
    baseURL: 'https://example.com',
    headless: true,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry'
  },

  retries: 2,

  workers: 4,

  reporter: 'html'
});
```

 ### Comandos essenciais para decorar do **Playwright**:

 | Comando | Descrição |
| --- | --- |
| `goto()` | Navega para uma URL específica. |
| `getByRole()` | Localiza um elemento com base em seu papel acessível (role), como `button`, `link`, `textbox`, etc. |
| `getByText()` | Localiza um elemento pelo texto visível. |
| `getByLabel()` | Localiza um elemento associado a um determinado `<label>`. |
| `getByTestId()` | Localiza um elemento pelo atributo de teste, geralmente `data-testid`. |
| `locator()` | Localiza elementos usando seletores CSS, XPath ou outros critérios. |
| `click()` | Clica em um elemento. |
| `fill()` | Preenche um campo de entrada com um determinado valor. |
| `press()` | Pressiona uma tecla específica em um elemento, como `Enter`, `Tab` ou `Escape`. |
| `check()` | Marca um checkbox ou radio button. |
| `uncheck()` | Desmarca um checkbox. |
| `selectOption()` | Seleciona uma opção em um elemento `<select>`. |
| `setInputFiles()` | Faz upload de um ou mais arquivos em um campo de seleção de arquivos. |
| `waitForURL()` | Aguarda a página navegar para uma URL que corresponda ao padrão especificado. |
| `waitForLoadState()` | Aguarda determinado estado de carregamento da página, como `load`, `domcontentloaded` ou `networkidle`. |
| `screenshot()` | Captura uma imagem da página ou de um elemento. |
| `expect()` | Faz uma asserção para verificar se o resultado obtido corresponde ao esperado. |
| `page.pause()` | Pausa a execução do teste e abre o modo de inspeção/debug do Playwright. |
## Avançado

[**Playwright Cheat Sheet avançado**](./avancado.md), cobrindo **Page Object Model, fixtures, autenticação, API testing, interceptação de requests, paralelismo e CI/CD**.