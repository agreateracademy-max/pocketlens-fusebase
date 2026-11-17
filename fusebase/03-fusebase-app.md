# 03 — FuseBase app

```bash
mkdir pocketlens-fusebase-app && cd pocketlens-fusebase-app
fusebase init            # org Sans Titre, product "PocketLens"
```

Copy the SPA in, keeping the scaffold's build config:

```bash
cp -R ../pocketlens-fusebase/web/src ./src
cp ../pocketlens-fusebase/web/index.html .
cp ../pocketlens-fusebase/web/tailwind.config.ts .
cp ../pocketlens-fusebase/web/postcss.config.js .
```

Keep the scaffold's `package.json`, `vite.config.ts` and `tsconfig`: they carry FuseBase's
base path, auth bootstrap and build output. Merge PocketLens dependencies into it:
`@supabase/supabase-js`, `@tanstack/react-query`, `react-router-dom`, `recharts`,
`framer-motion`, `sonner`, `next-themes`, `lucide-react`, and the Radix packages.

Create `.env` from `env.fusebase.example`, then:

```bash
fusebase dev start
fusebase deploy
fusebase app list
```

`fusebase app list` is the only reliable source for the live URL. Do not guess the subdomain.
