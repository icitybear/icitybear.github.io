---
title: "Test1" #标题
date: 2023-05-17T20:46:05+08:00 #创建时间
lastmod: 2023-05-17T20:46:05+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- 
tags: # 标签
-
keywords: 
- 
description: "" #描述 每个文章内容前面的展示描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论 有自带的扩展成twikoo
showToc: true # 显示目录 文章侧边栏toc目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false

# reward: true # 打赏
mermaid: true #自己加的是否开启mermaid
---
## 高亮模板
{{< highlight php >}}
class SafeProtocolService extends BaseService
{
    public const SAFE_ID_CHATROOM = '1_1_6_1_1';
    public const SAFE_ID_LIVE     = '1_1_5_1_1';

    public function checkWithSafeId($account_uuid = '', $safeId = '')
    {
        if (empty($account_uuid)) {
            return null;
        }

        if (empty($safeId)) {
            return null;
        }

        // 开关
        $config = c('risk_account_config');
        if (0 == $config['check_status']) {
            return null;
        }
        $soa = SoaClient::getSoa('j113_risk_safe_center', 'safeProtocol', 1, 2);

        $res = $soa->check([
            'appCode'    => b('appcode') ?: 1,
            'cloned'     => b('cloned') ?: 1,
            'safeId'     => $safeId,
            'senderUuid' => $account_uuid,
            'token'      => b('token'),
        ]);

        if ($soa->hasError()) {
            LOG::n('引导认证校验: account_uuid: ' . $account_uuid . ' , 系统错误, ' . json_encode($res, JSON_UNESCAPED_UNICODE));

            return null;
        }

        if ('OK' != $res['hitType']['code']) {
            // 版本判断
            $platform_id = b('platform_id');
            $cloned      = b('cloned');
            $app_version = b('app_version');
            if (!in_array($platform_id, [1, 2, 3])) {
                return $this->upgradeShowDialog($platform_id, $cloned);
            }
            if (1 == $platform_id && $app_version < $config['upgrade_version_limit']['android']) {
                return $this->upgradeShowDialog($platform_id, $cloned);
            }
            if (in_array($platform_id, [2, 3]) && $app_version < $config['upgrade_version_limit']['ios']) {
                return $this->upgradeShowDialog($platform_id, $cloned);
            }

            return $this->setError('account_risk_forbidden');
        }

        LOG::d('引导认证校验: account_uuid: ' . $account_uuid . ' , 结果, ' . json_encode($res, JSON_UNESCAPED_UNICODE));

        return null;
    }

    /**
     * 聊天室操作安全校验-引导认证校验
     *
     * @param string $account_uuid
     *
     * @return null|array
     */
    public function checkForChatRoom(string $account_uuid)
    {
        return $this->checkWithSafeId($account_uuid, self::SAFE_ID_CHATROOM);
    }

    /**
     * 直播操作安全校验-引导认证校验
     *
     * @param string $account_uuid
     *
     * @return null|array
     */
    public function checkForLive(string $account_uuid)
    {
        return $this->checkWithSafeId($account_uuid, self::SAFE_ID_LIVE);
    }

    /**
     * 低版本升级提示
     *
     * @param mixed $platform_id
     * @param mixed $cloned
     *
     * @return array
     */
    private function upgradeShowDialog($platform_id, $cloned)
    {
        if (1 == $platform_id) {
            $relation_list = c('update_version_relation.android');
            if (50 != $cloned && 7 != $cloned) {
                $cloned = 1;
            }
        } else {
            $cloned        = 1;
            $relation_list = c('update_version_relation.ios');
        }
        $relation = $relation_list[$cloned];

        $updateMsg = [
            'title'         => '更新提示',
            'content'       => '当前账号存在风险，请更新至最新版进行校验，以免影响您的正常交友。',
            'confirmText'   => '立即更新',
            'cancelText'    => '',
            'confirmSchema' => $relation ?? '',
            'cancelSchema'  => '',
        ];

        return $this->setError('show_dialog', json_encode($updateMsg, JSON_UNESCAPED_UNICODE));
    }
}
{{< /highlight >}}

## 无高亮模板

    class SafeProtocolService extends BaseService
    {
        public const SAFE_ID_CHATROOM = '1_1_6_1_1';
        public const SAFE_ID_LIVE     = '1_1_5_1_1';

        public function checkWithSafeId($account_uuid = '', $safeId = '')
        {
            if (empty($account_uuid)) {
                return null;
            }

            if (empty($safeId)) {
                return null;
            }

            // 开关
            $config = c('risk_account_config');
            if (0 == $config['check_status']) {
                return null;
            }
            $soa = SoaClient::getSoa('j113_risk_safe_center', 'safeProtocol', 1, 2);

            $res = $soa->check([
                'appCode'    => b('appcode') ?: 1,
                'cloned'     => b('cloned') ?: 1,
                'safeId'     => $safeId,
                'senderUuid' => $account_uuid,
                'token'      => b('token'),
            ]);

            if ($soa->hasError()) {
                LOG::n('引导认证校验: account_uuid: ' . $account_uuid . ' , 系统错误, ' . json_encode($res, JSON_UNESCAPED_UNICODE));

                return null;
            }

            if ('OK' != $res['hitType']['code']) {
                // 版本判断
                $platform_id = b('platform_id');
                $cloned      = b('cloned');
                $app_version = b('app_version');
                if (!in_array($platform_id, [1, 2, 3])) {
                    return $this->upgradeShowDialog($platform_id, $cloned);
                }
                if (1 == $platform_id && $app_version < $config['upgrade_version_limit']['android']) {
                    return $this->upgradeShowDialog($platform_id, $cloned);
                }
                if (in_array($platform_id, [2, 3]) && $app_version < $config['upgrade_version_limit']['ios']) {
                    return $this->upgradeShowDialog($platform_id, $cloned);
                }

                return $this->setError('account_risk_forbidden');
            }

            LOG::d('引导认证校验: account_uuid: ' . $account_uuid . ' , 结果, ' . json_encode($res, JSON_UNESCAPED_UNICODE));

            return null;
        }

        /**
        * 聊天室操作安全校验-引导认证校验
        *
        * @param string $account_uuid
        *
        * @return null|array
        */
        public function checkForChatRoom(string $account_uuid)
        {
            return $this->checkWithSafeId($account_uuid, self::SAFE_ID_CHATROOM);
        }

        /**
        * 直播操作安全校验-引导认证校验
        *
        * @param string $account_uuid
        *
        * @return null|array
        */
        public function checkForLive(string $account_uuid)
        {
            return $this->checkWithSafeId($account_uuid, self::SAFE_ID_LIVE);
        }

        /**
        * 低版本升级提示
        *
        * @param mixed $platform_id
        * @param mixed $cloned
        *
        * @return array
        */
        private function upgradeShowDialog($platform_id, $cloned)
        {
            if (1 == $platform_id) {
                $relation_list = c('update_version_relation.android');
                if (50 != $cloned && 7 != $cloned) {
                    $cloned = 1;
                }
            } else {
                $cloned        = 1;
                $relation_list = c('update_version_relation.ios');
            }
            $relation = $relation_list[$cloned];

            $updateMsg = [
                'title'         => '更新提示',
                'content'       => '当前账号存在风险，请更新至最新版进行校验，以免影响您的正常交友。',
                'confirmText'   => '立即更新',
                'cancelText'    => '',
                'confirmSchema' => $relation ?? '',
                'cancelSchema'  => '',
            ];

            return $this->setError('show_dialog', json_encode($updateMsg, JSON_UNESCAPED_UNICODE));
        }
    }


## todo