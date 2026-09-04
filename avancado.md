## Playwright Cheat Sheet — Avançado

### 1\. Page Object Model (POM)

Estrutura:

```
tests/
├── login.spec.ts
├── pages/
│   ├── LoginPage.ts
│   └── DashboardPage.ts
└── fixtures/
    └── test.ts
```

 Page Object:

```
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly email: Locator;
  readonly password: Locator;
  readonly loginButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.email = page.getByLabel('Email');
    this.password = page.getByLabel('Senha');
    this.loginButton = page.getByRole('button', { name: 'Entrar' });
  }

  async login(email: string, password: string) {
    await this.email.fill(email);
    await this.password.fill(password);
    await this.loginButton.click();
  }
}
```

 Teste:

```
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('login', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await page.goto('/login');

  await loginPage.login(
    'usuario@email.com',
    '123456'
  );

  await expect(page).toHaveURL(/dashboard/);
});
```

---

 ## 2\. Fixtures

 Crie objetos reutilizáveis para os testes:

```
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

type Fixtures = {
  loginPage: LoginPage;
};

export const test = base.extend<Fixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  }
});

export { expect } from '@playwright/test';
```

 Uso:

```
import { test, expect } from './fixtures/test';

test('login', async ({ page, loginPage }) => {
  await page.goto('/login');

  await loginPage.login(
    'usuario@email.com',
    '123456'
  );

  await expect(page).toHaveURL(/dashboard/);
});
```

---

 ## 3\. Autenticação

 Uma das melhores práticas é **não fazer login novamente em todos os testes**.

 Crie um setup:

```
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');

  await page.getByLabel('Email')
    .fill('usuario@email.com');

  await page.getByLabel('Senha')
    .fill('123456');

  await page.getByRole('button', {
    name: 'Entrar'
  }).click();

  await page.context().storageState({
    path: 'playwright/.auth/user.json'
  });
});
```

 No `playwright.config.ts`:

```
use: {
  storageState: 'playwright/.auth/user.json'
}
```

 Agora os testes começam já autenticados.

---

 ## 4\. API Testing

 GET:

```
const response = await request.get('/api/users');

expect(response.status()).toBe(200);

const body = await response.json();

expect(body).toHaveLength(10);
```

 POST:

```
const response = await request.post('/api/users', {
  data: {
    name: 'João',
    email: 'joao@email.com'
  }
});

expect(response.status()).toBe(201);
```

 PUT:

```
await request.put('/api/users/10', {
  data: {
    name: 'João Silva'
  }
});
```

 DELETE:

```
const response = await request.delete('/api/users/10');

expect(response.ok()).toBeTruthy();
```

 Headers:

```
await request.get('/api/users', {
  headers: {
    Authorization: `Bearer ${token}`
  }
});
```

---

 ## 5\. Interceptar Requests

 Ver uma requisição:

```
page.on('request', request => {
  console.log(
    request.method(),
    request.url()
  );
});
```

 Ver respostas:

```
page.on('response', response => {
  console.log(
    response.status(),
    response.url()
  );
});
```

 Interceptar:

```
await page.route('**/api/users', async route => {
  await route.continue();
});
```

 Mockar:

```
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify([
      {
        id: 1,
        name: 'João'
      }
    ])
  });
});
```

---

 ## 6\. Mockar erro da API

```
await page.route('**/api/users', async route => {
  await route.fulfill({
    status: 500,
    contentType: 'application/json',
    body: JSON.stringify({
      error: 'Internal Server Error'
    })
  });
});
```

 Teste:

```
await expect(
  page.getByText('Erro ao carregar usuários')
).toBeVisible();
```

---

 ## 7\. Esperar uma API

 Muito útil:

```
const responsePromise =
  page.waitForResponse('**/api/users');

await page.getByRole('button', {
  name: 'Carregar'
}).click();

const response = await responsePromise;

expect(response.status()).toBe(200);
```

 Também:

```
const requestPromise =
  page.waitForRequest('**/api/users');

await page.getByRole('button', {
  name: 'Salvar'
}).click();

const request = await requestPromise;

console.log(request.method());
```

---

 ## 8\. Downloads

```
const downloadPromise =
  page.waitForEvent('download');

await page.getByText('Baixar relatório').click();

const download = await downloadPromise;

await download.saveAs(
  'downloads/relatorio.pdf'
);
```

 Ver nome:

```
console.log(download.suggestedFilename());
```

---

 ## 9\. Upload

 Arquivo único:

```
await page
  .getByLabel('Arquivo')
  .setInputFiles('arquivo.pdf');
```

 Múltiplos:

```
await page
  .locator('input[type="file"]')
  .setInputFiles([
    'arquivo1.pdf',
    'arquivo2.pdf'
  ]);
```

---

 ## 10\. Nova aba

```
const popupPromise =
  page.waitForEvent('popup');

await page.getByText('Abrir documento').click();

const popup = await popupPromise;

await popup.waitForLoadState();

console.log(await popup.title());
```

---

 ## 11\. Múltiplas páginas

```
const pages = context.pages();

console.log(pages.length);
```

 Criar nova página:

```
const newPage = await context.newPage();

await newPage.goto('/dashboard');
```

---

 ## 12\. Frames

```
const frame = page.frameLocator(
  '#payment-frame'
);

await frame.getByLabel('Card number')
  .fill('4111111111111111');
```

---

 ## 13\. Localizadores avançados

 Por texto:

```
page.getByText('Comprar');
```

 Por role:

```
page.getByRole('button', {
  name: 'Comprar'
});
```

 Por label:

```
page.getByLabel('Email');
```

 Por placeholder:

```
page.getByPlaceholder(
  'Digite seu email'
);
```

 Por test ID:

```
page.getByTestId('login-button');
```

 CSS:

```
page.locator('.login-button');
```

 Filtrar:

```
page.locator('.card').filter({
  hasText: 'Playwright'
});
```

 Primeiro:

```
page.locator('.item').first();
```

 Último:

```
page.locator('.item').last();
```

 N-ésimo:

```
page.locator('.item').nth(2);
```

---

 ## 14\. Assertions avançadas

```
await expect(page).toHaveURL('/dashboard');

await expect(page).toHaveTitle(
  'Dashboard'
);

await expect(locator).toBeVisible();

await expect(locator).toBeHidden();

await expect(locator).toBeEnabled();

await expect(locator).toBeDisabled();

await expect(locator).toBeChecked();

await expect(locator).toHaveText('Olá');

await expect(locator).toContainText('Olá');

await expect(locator).toHaveValue('João');

await expect(locator).toHaveAttribute(
  'href',
  '/dashboard'
);

await expect(locator).toHaveCount(5);
```

---

 ## 15\. Testes parametrizados

```
const users = [
  {
    email: 'admin@test.com',
    password: '123'
  },
  {
    email: 'user@test.com',
    password: '456'
  }
];

for (const user of users) {
  test(`login ${user.email}`, async ({ page }) => {
    await page.goto('/login');

    await page.getByLabel('Email')
      .fill(user.email);

    await page.getByLabel('Senha')
      .fill(user.password);

    await page.getByRole('button', {
      name: 'Entrar'
    }).click();
  });
}
```

---

 ## 16\. `test.describe`

```
test.describe('Login', () => {

  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('login válido', async ({ page }) => {
    // ...
  });

  test('senha inválida', async ({ page }) => {
    // ...
  });

});
```

 Hooks disponíveis:

```
test.beforeAll()
test.afterAll()

test.beforeEach()
test.afterEach()
```

---

 ## 17\. Tags

```
test(
  'login @smoke',
  async ({ page }) => {
    // ...
  }
);
```

 Executar:

```
npx playwright test --grep @smoke
```

 Excluir:

```
npx playwright test --grep-invert @slow
```

---

 ## 18\. Projetos / Browsers

 No config:

```
projects: [
  {
    name: 'chromium',
    use: {
      browserName: 'chromium'
    }
  },
  {
    name: 'firefox',
    use: {
      browserName: 'firefox'
    }
  },
  {
    name: 'webkit',
    use: {
      browserName: 'webkit'
    }
  }
]
```

 Executar:

```
npx playwright test --project=chromium
```

---

 ## 19\. Mobile

```
projects: [
  {
    name: 'mobile',
    use: {
      ...devices['iPhone 13']
    }
  }
]
```

 Ou:

```
import { devices } from '@playwright/test';
```

---

 ## 20\. Paralelismo

 Configuração:

```
workers: 4
```

 Forçar sequência:

```
fullyParallel: false
```

 Executar com um worker:

```
npx playwright test --workers=1
```

---

 ## 21\. Retries

```
retries: 2
```

 Ou:

```
npx playwright test --retries=2
```

---

 ## 22\. Trace

 No config:

```
use: {
  trace: 'on-first-retry'
}
```

 Ou:

```
npx playwright show-trace trace.zip
```

 O trace permite investigar:

 - ações executadas;
- DOM;
- screenshots;
- requests;
- console;
- tempo de cada operação.

---

 ## 23\. Screenshots e vídeos

```
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure'
}
```

 Screenshot manual:

```
await page.screenshot({
  path: 'evidence.png',
  fullPage: true
});
```

---

 ## 24\. Console do navegador

```
page.on('console', message => {
  console.log(message.text());
});
```

 Erros:

```
page.on('pageerror', error => {
  console.log(error.message);
});
```

---

 ## 25\. Geolocation

```
const context = await browser.newContext({
  geolocation: {
    latitude: -22.9068,
    longitude: -43.1729
  },
  permissions: ['geolocation']
});
```

---

 ## 26\. Viewport

```
await page.setViewportSize({
  width: 1920,
  height: 1080
});
```

---

 ## 27\. Variáveis de ambiente

 `.env`:

```
BASE_URL=https://example.com
USER_EMAIL=test@example.com
```

 Uso:

```
const email = process.env.USER_EMAIL;
```

 No config:

```
use: {
  baseURL: process.env.BASE_URL
}
```

---

 ## 28\. CI/CD

 Comandos típicos:

```
npm ci

npx playwright install --with-deps

npx playwright test

npx playwright show-report
```

 Um pipeline normalmente faz:

```
Install
   ↓
Install browsers
   ↓
Run tests
   ↓
Collect artifacts
   ↓
Publish report
```

---

 ## 29\. Comandos que vale decorar

 ### 🟢 Essenciais

```
page.goto()
page.getByRole()
page.getByLabel()
page.getByText()
page.getByTestId()
locator.click()
locator.fill()
locator.press()
expect()
```

 ### 🟡 Intermediários

```
waitForURL()
waitForResponse()
waitForRequest()
waitForEvent()
setInputFiles()
selectOption()
check()
uncheck()
route()
```

 ### 🔴 Avançados

```
storageState()
browser.newContext()
context.newPage()
page.frameLocator()
page.route()
request.get()
request.post()
test.extend()
test.beforeEach()
test.beforeAll()
```

 ## 30\. Estrutura profissional recomendada

```
playwright-project/
│
├── tests/
│   ├── login.spec.ts
│   ├── users.spec.ts
│   └── checkout.spec.ts
│
├── pages/
│   ├── LoginPage.ts
│   ├── UsersPage.ts
│   └── CheckoutPage.ts
│
├── fixtures/
│   └── test.ts
│
├── utils/
│   ├── api.ts
│   └── helpers.ts
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

 **Regra de ouro:** mantenha os testes focados no **comportamento**, coloque a interação com a página nos **Page Objects**, dados reutilizáveis nas **fixtures/utils** e evite `waitForTimeout()` sempre que houver uma condição real que você possa aguardar.