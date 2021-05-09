Initial setup:

```
mkdir example
cd example
npm install --save-dev webpack webpack-cli typescript ts-loader http-server
npm install --save sql.js-httpvfs
npx tsc --init

#Edit the generated tsconfig.json file to make it more modern:
#
#...
#"target": "es2020",
#"module": "esnext",
#"moduleResolution": "node",
#...

#Create a webpack config, minimal index.html file and TypeScript entry point:

cat > webpack.config.js << 'END'
module.exports = {
  entry: "./src/index.ts",
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: "ts-loader",
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: [".tsx", ".ts", ".js"],
  },
  output: {
    filename: "bundle.js",
  },
  devServer: {
    publicPath: "/dist",
  },
};
END
cat > index.html << END
<!DOCTYPE html>

<title>Minimal sql.js-httpvfs demo</title>
<script src="./dist/bundle.js"></script>
<div>Hello World!</div>
END
mkdir src
car > src/index.ts << END
import { createDbWorker } from "sql.js-httpvfs";
const workerUrl = new URL(
  "sql.js-httpvfs/dist/sqlite.worker.js",
  import.meta.url
);
const wasmUrl = new URL("sql.js-httpvfs/dist/sql-wasm.wasm", import.meta.url);
async function load() {
  const worker = await createDbWorker(
    [
      {
        from: "inline",
        config: {
          serverMode: "full",
          url: "/example.sqlite3",
          requestChunkSize: 4096,
        },
      },
    ],
    workerUrl.toString(),
    wasmUrl.toString()
  );
  const result = await worker.db.query(`select * from mytable`);
  document.body.textContent = JSON.stringify(result);
}
load();
END

#Finally, create a database:
sqlite3 example.sqlite3 "create table mytable(foo, bar)"
sqlite3 example.sqlite3 "insert into mytable values ('hello', 'world')"

#and build the JS bundle and start a webserver:

~/node_modules/.bin/webpack --mode=development
~/node_modules/.bin/http-server
```
