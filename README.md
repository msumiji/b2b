*This project has been created as part of the 42 curriculum by msumiji*

# Description
## OS selection
This assignment requires the use of either Debian or Rocky as the operating system. For this project, Debian was used for the following reasons.
 Intuitive and easy to use.
 Low memory usage.
 Easy for reviewers to use as well.

### 1. Simple Firewall Configuration
Firewall configuration is required on Born2beroot, but the tools used differ.<br>
* Debian: We use ufw (Uncomplicated Firewall). Configuration is straightforward using commands that are easy for humans to read and understand, such as `ufw allow 4242`.
* Rocky Linux: We use firewalld (firewall-cmd). There are more conventions to remember, such as the concept of zones, the `--permanent` option, and the need to remember to run `--reload`.

### 2. Simplicity of Password Policy Configuration (PAM)
In this exercise, you are required to configure very detailed password policies—such as password expiration, character count, case sensitivity, restrictions on consecutive characters, and prohibitions on including the username—using PAM (Pluggable Authentication Modules).

* Debian: Simply install `libpam-pwquality` and list the parameters in a single file, `/etc/pam.d/common-password`; the changes are applied intuitively.
* Rocky Linux: Because `authselect`—a system-wide authentication profile management tool—is involved, the process is cumbersome: editing configuration files directly may result in them being overwritten, or you may need to create custom profiles.

### 3. Lightweight and low on resource consumption (ideal for VirtualBox)
For this assignment, you’ll install an OS on a virtual machine (VM) running on VirtualBox, but it’s assumed that you’ll be running it on the school’s iMacs or laptops, which have limited memory and storage.
* Debian: When installed in a minimal configuration (netinst without a GUI), both disk usage and memory consumption are extremely low, leaving plenty of leeway for capacity calculations even with LVM partitioning (especially the detailed partitioning in the Bonus section).
* Rocky Linux: Even in a minimal configuration, its base size is larger than Debian’s, and the package manager (dnf) consumes more memory.

### 4. AppArmor's easy of use
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

### VirtualBox vs UTM

In Born2beRoot, **VirtualBox** is the primary virtualization tool, while **UTM** is permitted as an alternative when VirtualBox cannot be used[cite: 4, 7].

| Feature | VirtualBox | UTM |
| :--- | :--- | :--- |
| **Role in Project** | Primary / default choice[cite: 4, 7] | Alternative when VirtualBox cannot be used[cite: 4, 7] |
| **Host Platforms** | Windows, macOS, Linux[cite: 17] | macOS (including Apple Silicon setups)[cite: 17] |
| **Execution Approach** | Hardware-assisted virtualization | QEMU-backed (virtualization and emulation) |
| **Virtual Disk Format** | `.vdi`[cite: 17] | `.qcow2`[cite: 17] |
| **Typical VM Path** | `~/VirtualBox VMs/` or `%HOMEDRIVE%%HOMEPATH%\VirtualBox VMs\`[cite: 17] | `~/Library/Containers/com.utmapp.UTM/Data/Documents/` (e.g., Mac M1)[cite: 17] |

* **VirtualBox:** Widely supported across desktop operating systems and well-aligned with standard Born2beRoot workflows, making disk image management and SHA-1 signature generation straightforward[cite: 4, 7, 17].
* **UTM:** Useful when VirtualBox is unavailable, leveraging QEMU for virtualization and emulation[cite: 4, 7]. It uses `.qcow2` disk images with different storage paths and hashing commands that must be accounted for when generating `signature.txt`[cite: 17].

### Logical Volume Manager (LVM)

**LVM (Logical Volume Manager)** abstracts physical disks or partitions into flexible **Logical Volumes (LVs)** instead of managing raw partitions directly. This makes resizing storage, grouping multiple areas, and managing snapshots easier than with traditional fixed partitions.

#### Core Components
- **PV (Physical Volume)**: The underlying disk or partition used by LVM (e.g., an encrypted partition mapped via LUKS such as `sda5_crypt`).
- **VG (Volume Group)**: A storage pool combining one or more PVs into a single manageable space.
- **LV (Logical Volume)**: Virtual partitions allocated from a VG where filesystems (such as `ext4` or `xfs`) or swap space are created and mounted.

#### Role in Born2beRoot
- Encrypted partitions are set up with LVM on top (for example, an encrypted container mapped as `sda5_crypt` backing a volume group with logical volumes for `/`, swap, `/home`, etc.).
- LVM status is also reported by `monitoring.sh` (e.g., `#LVM use: yes`).<br>

## Secure Shell (SSH)

**SSH (Secure Shell)** is a network protocol and toolset for securely connecting to remote machines (such as servers or virtual machines) over a network to run commands and transfer files over an encrypted channel. Unlike unencrypted protocols (such as Telnet or FTP), encryption helps protect credentials, commands, and transferred data from eavesdropping or tampering.

### Key Concepts
- **Encrypted Channel & Host Verification**: Establishes encrypted communication via key exchange and verifies server identity using host keys / fingerprints.
- **Authentication Methods**: Supports password authentication and public-key authentication (using a key pair).
- **Port Configuration**: Uses port `22` by default, though it can be configured to listen on a custom port.
- **Common Tools**: Includes `ssh` for remote shell access and `scp` / `sftp` for file transfers over SSH.

### Role in Born2beRoot
- The SSH daemon must run on port `4242` with direct `root` login disabled[cite: 9].
- Port `4242` must be allowed through the firewall (`ufw` or `firewalld`)[cite: 9, 12].
- SSH behavior and configuration may be tested during evaluation (for instance, with a newly created user account)[cite: 9].<br><br>

# Instructions

## 1. Prerequisites
- Install **VirtualBox** (or **UTM**) on the host machine[cite: 4, 7].
- Download the latest stable ISO image for **Debian**[cite: 8].

## 2. Virtual Machine & OS Installation
1. Create a new virtual machine in VirtualBox (no GUI / desktop environment)[cite: 4, 7, 8].
2. Boot from the ISO and install the OS using encrypted partitions with **LVM** (e.g., an encrypted container mapped as `sda5_crypt` with logical volumes for `/`, swap, and `/home`)[cite: 8, 9].
3. Create the standard user with your login name and add it to the `user42` and `sudo` groups:
   ```bash
   usermod -aG user42,sudo <login>

## 3. Security & System Configuration
1. SH (port 4242, no root login): Edit /etc/ssh/sshd_config to set Port 4242 and PermitRootLogin no, then restart the SSH service[cite: 9].
2. Firewall: Enable the firewall and allow only port 4242 for Debian(ufw) (mandatory part):
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 4242/tcp
sudo ufw enable
```
3. sudo policy: Configure sudo rules (e.g., via sudo visudo or a file under /etc/sudoers.d/) to enforce:
* Maximum of 3 authentication attempts (passwd_tries=3)
* Custom error message on failed authentication (badpass_message="...")
* Input and output logging under /var/log/sudo/ (log_input, log_output, iolog_dir=/var/log/sudo/)
* requiretty enabled
* Restricted secure_path
4. Password policy:
Configure password aging (e.g., /etc/login.defs and chage) and password quality rules (e.g., PAM pam_pwquality.so), then update passwords for root and existing users after applying the rules.

## 4. Installing and Scheduling
1. Copy monitoring.sh to /usr/local/bin/monitoring.sh and grant execute permission:
```bash
sudo cp monitoring.sh /usr/local/bin/monitoring.sh
sudo chmod 755 /usr/local/bin/monitoring.sh
```
2. Open root's crontab and schedule execution at reboot and every 10 minutes (using wall inside the script to broadcast to all terminals):
```bash
sudo crontab -e
```
Add:
```
@reboot /usr/local/bin/monitoring.sh
*/10 * * * * /usr/local/bin/monitoring.sh
```
3. Run the script manually to verify output without errors:
```bash
sudo /usr/local/bin/monitoring.sh
```
## 5. Verification & Repository Preparation
* Check listening ports and firewall status (e.g., ss -tunlp and sudo ufw status or sudo firewall-cmd --list-all).
* Generate the SHA-1 signature of the virtual disk file (.vdi or .qcow2) and save it to signature.txt at the repository root without committing the VM disk itself.
* emove all VM snapshots before evaluation begins.


snapshot
UFW
sudo
hostname


# 説明
## OSの選択
　本課題では、OSとしてDebianあるいはRockyを使うことが要求されています。本プロジェクトでは、以下の理由からDebianを利用しました。<br>

### 1. ファイアウォール設定がシンプル
Born2berootではファイアウォールの設定が必須ですが、使用するツールが異なります。<br>
* Debian: ufw（Uncomplicated Firewall）を使います。ufw allow 4242 のように人間が読んでそのまま意味がわかるコマンドで完結します。<br>
* Rocky Linux: firewalld（firewall-cmd）を使います。ゾーン（zone）の概念や --permanent オプション、--reload の適用忘れなど、覚える作法が多くなります

### 2. パスワードポリシー設定（PAM）の簡潔さ
課題では「パスワードの有効期限・文字数・大文字小文字・連続文字制限・ユーザー名を含む禁止」など、非常に細かいパスワードポリシーをPAM（Pluggable Authentication Modules）で設定させられます。

* Debian: libpam-pwquality をインストールし、/etc/pam.d/common-password の1ファイルにパラメータを書き並べるだけで直感的に反映されます。
* Rocky Linux: authselect というシステム全体の認証プロファイル管理ツールが介入してくるため、設定ファイルを直接編集すると上書きされたり、カスタムプロファイルを作る必要があったりと手順が煩雑です。

### 3. 軽量でリソース消費が少ない（VirtualBox向き）
課題ではVirtualBox上の仮想マシン（VM）にOSをインストールしますが、学校のiMacやノートPCの限られたメモリ・ストレージ環境で動かす前提があります。
* Debian: 最小構成（netinstでGUIなし）でインストールすると、ディスク使用量もメモリ消費量も極めて少なく、LVMのパーティション分割（特にBonusの細かいパーティション分け）でも容量計算に余裕が生まれます。
* Rocky Linux: 最小構成でもDebianよりベースサイズが大きく、パッケージ管理（dnf）のメモリ消費量も多めです。

### 4. AppArmorの使いやすさ
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
## LVM（論理ボリュームマネージャ）の概要

**LVM (Logical Volume Manager)** は、物理ディスクやパーティションを直接扱う代わりに、サイズ変更や管理が柔軟な **論理ボリューム（LV）** として抽象化する仕組みです。固定パーティションよりも容量調整や領域管理を行いやすくなります。

#### 基本要素
- **PV（Physical Volume / 物理ボリューム）**: LVMの土台となるディスクやパーティション（LUKS等で暗号化した `sda5_crypt` などのマッピング対象も含む）。
- **VG（Volume Group / ボリュームグループ）**: 1つまたは複数のPVをまとめた記憶領域のプール。
- **LV（Logical Volume / 論理ボリューム）**: VGから切り出し、ファイルシステム（`ext4` や `xfs` など）やスワップ領域を割り当てる仮想的なパーティション。

#### Born2beRoot での位置づけ
- 暗号化パーティションの上にLVMを構成し、用途ごとの論理ボリューム（`/`、スワップ、`/home` など）を割り当てます。
- `monitoring.sh` でもLVMの利用状況（`#LVM use: yes` など）を出力します。

## VirtualBox vs UTM

Born2beRoot では仮想マシン（VM）を構築するツールとして **VirtualBox** が基本とされており、VirtualBox を利用できない環境向けの代替手段として **UTM** が認められています[cite: 4, 7]。

| 比較項目 | VirtualBox | UTM |
| :--- | :--- | :--- |
| **課題での位置づけ** | 基本・第一候補[cite: 4, 7] | VirtualBox が使えない場合の代替[cite: 4, 7] |
| **対応ホストOS** | Windows, macOS, Linux など幅広い環境[cite: 17] | macOS（および Apple Silicon 環境等）向け[cite: 17] |
| **仮想化・動作方式** | ハードウェア仮想化支援機能を活用した仮想化が中心 | QEMU 基盤（仮想化およびエミュレーション） |
| **仮想ディスク形式** | `.vdi` など[cite: 17] | `.qcow2` など[cite: 17] |
| **ディスク保存場所の例** | `~/VirtualBox VMs/`（Linux/macOS）や `%HOMEDRIVE%%HOMEPATH%\VirtualBox VMs\`（Windows）[cite: 17] | `~/Library/Containers/com.utmapp.UTM/Data/Documents/`（Mac M1 等）[cite: 17] |

* **VirtualBox の特徴:** マルチプラットフォームで扱いやすく、課題の標準的な前提環境として情報が豊富です[cite: 4, 7, 17]。仮想ディスクの SHA-1 ハッシュ値を取得して `signature.txt` に記録する運用もシンプルです[cite: 17]。
* **UTM の特徴:** VirtualBox が使用できない環境での有力な選択肢で、QEMU をベースに仮想化やアーキテクチャのエミュレーションを行えます[cite: 4, 7]。ディスクファイル（`.qcow2`）のパスやハッシュ取得手順が VirtualBox と異なるため、提出時の確認が必要です[cite: 17]。

## SSH（Secure Shell）の概要

**SSH (Secure Shell)** は、ネットワーク経由でリモートマシン（サーバーや仮想マシンなど）へ安全に接続し、暗号化された通信路上でコマンド実行やファイル転送を行うためのプロトコルおよびツール群です。平文通信を行う古い方式（Telnet や FTP など）と異なり、認証情報やコマンド、転送データを暗号化することで盗聴や改ざんのリスクを抑えます。

### 主な要素
- **暗号化通信とホスト検証**: 鍵交換によって通信を暗号化し、サーバー側の鍵やフィンガープリントによって接続先の真正性を確認します。
- **認証方式**: パスワード認証や、鍵ペアを用いる公開鍵認証などを利用できます。
- **ポート設定**: 標準ではポート `22` を使いますが、設定によって任意のポート番号へ変更できます。
- **主なツール**: リモート操作を行う `ssh` や、SSH経由でファイル転送を行う `scp` / `sftp` などがあります。

### Born2beRoot での位置づけ
- SSHサービスをポート `4242` で稼働させ、`root` ユーザーでの直接SSHログインを禁止します[cite: 9]。
- ファイアウォール（`ufw` または `firewalld`）でポート `4242` を許可します[cite: 9, 12]。
- 評価（defense）では、新規ユーザー作成などを通じてSSHの仕組みや動作が確認されます[cite: 9]。<br><br>

# 手順

## 1. 前提条件
- ホストマシンに **VirtualBox**（または **UTM**）をインストールする[出典: 4, 7]。
- **Debian**の最新の安定版ISOイメージをダウンロードする[出典: 8]。

## 2. 仮想マシンとOSのインストール
1. VirtualBoxで新しい仮想マシンを作成する（GUI／デスクトップ環境なし）[出典: 4, 7, 8]。
2. ISO イメージから起動し、**LVM** を使用した暗号化パーティションで OS をインストールします（例：`sda5_crypt` としてマッピングされた暗号化コンテナに、`/`、スワップ、および `/home` 用の論理ボリュームを設定）[出典: 8, 9]。
3. ログイン名を使用して標準ユーザーを作成し、そのユーザーを `user42` および `sudo` グループに追加します：
   ```bash
   usermod -aG user42,sudo <ログイン名>

## 3. セキュリティとシステム設定
1. SSH（ポート 4242、root ログイン不可）：/etc/ssh/sshd_config を編集して、ポート 4242 を指定し、PermitRootLogin を no に設定した後、SSH サービスを再起動します[出典: 9]。
2. ファイアウォール：ファイアウォールを有効にし、Debian（ufw）ではポート 4242 のみを許可します（必須項目）：
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 4242/tcp
sudo ufw enable
```
3. sudo ポリシー：sudo ルールを設定し（例：sudo visudo または /etc/sudoers.d/ 配下のファイルを使用）、以下を適用します：
* 認証試行回数を最大 3 回に制限（passwd_tries=3）
* 認証失敗時のカスタムエラーメッセージ（badpass_message=「...」）
* /var/log/sudo/ への入力および出力ログの記録 (log_input, log_output, iolog_dir=/var/log/sudo/)
* requiretty を有効化
* secure_path を制限
4. パスワードポリシー:
パスワードの有効期限設定（例: /etc/login.defs および chage）とパスワード品質ルール（例: PAM pam_pwquality.so）を設定し、ルールを適用した後、root および既存のユーザーのパスワードを更新します。

## 4. インストールとスケジュール設定
1. monitoring.sh を /usr/local/bin/monitoring.sh にコピーし、実行権限を付与します：
```bash
sudo cp monitoring.sh /usr/local/bin/monitoring.sh
sudo chmod 755 /usr/local/bin/monitoring.sh
```
2. root の crontab を開き、再起動時および 10 分ごとに実行されるようスケジュール設定します（スクリプト内で `wall` コマンドを使用して、すべての端末に通知します）：
```bash
sudo crontab -e
```
以下を追加します：
```
@reboot /usr/local/bin/monitoring.sh
*/10 * * * * /usr/local/bin/monitoring.sh
```
3. スクリプトを手動で実行し、エラーのない出力が得られることを確認します：
```bash
sudo /usr/local/bin/monitoring.sh
```
## 5. 検証とリポジトリの準備
* リスニングポートとファイアウォールの状態を確認します（例：ss -tunlp、sudo ufw status、または sudo firewall-cmd --list-all）。
* 仮想ディスクファイル（.vdi または .qcow2）の SHA-1 署名を生成し、VM ディスク自体をコミットせずに、リポジトリのルートディレクトリにある signature.txt に保存します。
* 評価開始前に、すべての VM スナップショットを削除してください。
