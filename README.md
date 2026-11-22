*This project has been created as part of the 42 curriculum by msumiji*

# Description
## OS selection
This assignment requires the use of either Debian or Rocky as the operating system. For this project, Debian was used for the following reasons.
 Intuitive and easy to use.
 Low memory usage.
 Easy for reviewers to use as well.

### Simple Firewall Configuration
Firewall configuration is required on Born2beroot, but the tools used differ.<br>
* Debian: We use ufw (Uncomplicated Firewall). Configuration is straightforward using commands that are easy for humans to read and understand, such as `ufw allow 4242`.
* Rocky Linux: We use firewalld (firewall-cmd). There are more conventions to remember, such as the concept of zones, the `--permanent` option, and the need to remember to run `--reload`.

### AppArmor's easy of use
Both AppArmor and SELinux (Security-Enhanced Linux) are Mandatory Access Control (MAC) security modules built into the Linux kernel. While traditional Linux permissions rely on Discretionary Access Control (DAC), MAC adds an extra kernel-level layer that restricts what processes and programs are allowed to access, limiting the impact if a service is compromised.

| Feature | AppArmor | SELinux |
| :--- | :--- | :--- |
| **Access Control Model** | **Path-based** (per-program profiles) | **Label-based** (security contexts / Type Enforcement) |
| **How It Works** | Binds profiles to specific executable paths to define allowed file paths and capabilities | Assigns security labels to all objects (files, processes, ports) and enforces strict policy rules between types |
| **Ease of Use & Learning Curve** | Rules use human-readable file paths, making configuration and debugging relatively straightforward | Highly granular and strict, but complex with a steeper learning curve |
| **Default Distributions** | Debian, Ubuntu | RHEL, Rocky Linux, Fedora |

* **AppArmor:** Focuses on individual applications and file paths. It is easier to read, configure, and troubleshoot—making it well-suited for straightforward server setups—though rules depend on file paths rather than underlying filesystem objects.
* **SELinux:** Enforces system-wide security contexts across files, processes, and ports. It offers finer-grained isolation regardless of file paths, but requires careful label and policy management (for example, adjusting port contexts when changing the SSH port).

Since Apparmor is easier to use, I chose a Debian distribution that supports it.

To check if Apparmor is running:
```bash
sudo systemctl status apparmor
```


snapshot
LVM
difference between Debian and Rocky linux
SSH
UFW
sudo
hostname


# 説明
## OSの選択
　本課題では、OSとしてDebianあるいはRockyを使うことが要求されています。本プロジェクトでは、以下の理由からDebianを利用しました。<br>

### ファイアウォール設定がシンプル
Born2berootではファイアウォールの設定が必須ですが、使用するツールが異なります。<br>
* Debian: ufw（Uncomplicated Firewall）を使います。ufw allow 4242 のように人間が読んでそのまま意味がわかるコマンドで完結します。<br>
* Rocky Linux: firewalld（firewall-cmd）を使います。ゾーン（zone）の概念や --permanent オプション、--reload の適用忘れなど、覚える作法が多くなります

### メモリ消費量が少ない

### AppArmorの使いやすさ
AppArmor と SELinux は、どちらも Linux カーネルにおける**強制アクセス制御（MAC: Mandatory Access Control）**の仕組みです。通常の Linux のファイル権限（DAC）に加え、プログラムやプロセスが許可されたリソース以外にアクスするのをカーネルレベルで制限し、万が一サービスが侵害された場合の被害拡大を防ぎます。

| 比較項目 | AppArmor | SELinux |
| :--- | :--- | :--- |
| **制御の基準** | **ファイルパス（プログラム）ベース** | **セキュリティラベル（コンテキスト）ベース** |
| **仕組み** | 実行ファイルごとにプロファイルを紐づけ、アクセス可能なファイルパスや権限を定義する | ファイル・プロセス・ポートなど全対象にラベル（Type等）を付与し、ポリシーで許可された組み合わせのみを通す |
| **学習・管理のしやすさ** | ルールがパス表記で直感的なため、設定やトラブルシューティングが比較的容易 | 非常に厳格で細かい制御が可能だが、構造が複雑で学習コスト・運用負荷が高い |
| **主な採用ディストリビューション** | Debian, Ubuntu | RHEL, Rocky Linux, Fedora |

* **AppArmor の特徴:** プログラム単位で「どのパスを読み書きしてよいか」を記述するため可読性が高く、システム管理の初学者でも扱いやすい点がメリットです。一方で、シンボリックリンクやパスの移動・マウント構成の変化には注意が必要です。
* **SELinux の特徴:** ファイルやプロセス、ネットワークポートまで一貫したラベル管理（Type Enforcement など）を行うため、パス変更の影響を受けにくく極めて堅牢です。ただし、SSHのポート変更や独自ディレクトリの追加時にもラベルやポリシーの調整が必要になるなど、設定の難易度が高くなります。
　このように、Apparmorの方が扱いやすいため、Apparmorが使えるDebianを選びました。

Apparmorが起動しているか確認するには
```bash
sudo systemctl status apparmor
```
