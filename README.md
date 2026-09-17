# responsibility-driven-crates

Rust の crate / workspace 構成を、あらかじめ決めたレイヤーではなく、実際に存在する責務に沿って設計するための軽量スキルです。

関連する責務はまずまとめ、責務が独立した API・依存関係・consumer・テスト・ビルド要件などを持つようになった段階で crate 分離を検討します。

crate 名は `core`、`common`、`shared` のような位置づけだけを示す名前より、`auth`、`storage`、`api` など実際の責務を直接表す名前を優先します。

workspace 内のディレクトリ名と Cargo package 名も分けて考えます。
たとえばディレクトリは `crates/auth/`、公開 package は `my-project-auth` とする構成を扱います。

実際の規則は [`SKILL.md`](./SKILL.md) を参照してください。
